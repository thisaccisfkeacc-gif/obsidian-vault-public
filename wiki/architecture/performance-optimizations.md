# Performance Optimizations — Macro Editor Hot Paths

> **Date:** 2026-08-05
> **Scope:** MacroEditorViewModel (Properties, Core, Commands) + UndoRedoService
> **Impact:** Move Up/Down, drag-drop, property edits, display rebuilds, undo/redo

---

## Why This Document Exists

These optimizations touch fundamental data-flow paths. If a new bug appears in the editor (wrong undo state, stale display, missing validation, move/drag glitches), **check this document first** — the fix probably interacted with one of these changes.

---

## Quick Reference — All 6 Fixes

| # | What | File(s) | Risk Level | What Could Break |
|---|------|---------|------------|------------------|
| 1 | Debounce clone skip | `UndoRedoService.cs` | Low | Undo stack missing a state if signature collision |
| 2 | Display rebuild skip | `MacroEditorViewModel.Properties.cs` | Medium | Stale timeline if version counter drifts |
| 3 | StepsEqual fast reject | `UndoRedoService.cs` | Low | False equality if child[0] matches but rest differ |
| 4 | MoveSelectedSteps O(N²) | `MacroEditorViewModel.Core.cs` | Medium | Wrong move order if parent map misses nested steps |
| 5 | Single-pass metadata | `MacroEditorViewModel.Properties.cs` | Medium | Missing error count / named blocks / variable names |
| 6 | Duplicate names O(N) | `MacroEditorViewModel.Properties.cs` | Low | False duplicate flag if dictionary logic is wrong |

---

## Fix #1 — Debounce Clone Skip

### Problem
When a user edits a step property (e.g. toggles `IsDisabled` on then off), the value-change debounce timer (300ms) fires and calls `DeepCloneSteps()` to push an undo snapshot — even though the value reverted to the original. This wastes ~3-6ms on a 1000-step macro for no reason.

### Solution
- `_pendingValueChangeSignature`: captured when the user first edits a property in a batch. Computed from the watched steps at that moment.
- `_lastCommittedSignature`: updated after every successful clone.
- **Tick handler** (fires after 300ms debounce): computes current signature. If it equals `_pendingValueChangeSignature` → nothing actually changed → skip clone, skip push.
- **FlushPendingChanges**: same check before cloning.
- **StopWatching**: same check before flushing.

### Key Code
```
UndoRedoService.cs:
- _pendingValueChangeSignature (field)
- _lastCommittedSignature (field)
- OnStepPropertyChanged: captures signature on first edit
- Tick handler: signature comparison → skip
- FlushPendingChanges: signature comparison → skip
- StopWatching: signature comparison → skip
```

### What Could Go Wrong
- **Signature collision**: If two completely different step states produce the same signature, a real change could be skipped. The signature includes Id + Type + StepName + Value + child counts — very unlikely to collide, but possible if only deep nested properties changed.
- **Debug symptom**: User makes a real edit, but undo doesn't show it. Check `DebugLogger` for `VALUE_CHANGE skipped (reverted)` when it shouldn't have skipped.

---

## Fix #2 — Display Rebuild Skip

### Problem
`RefreshDisplaySteps()` was called on every `GlobalStepChanged` event, even when the step structure hadn't changed (only metadata like `IsDuplicateName` or `ErrorCount`). Each call creates a `Task.Run`, snapshots the collection on the UI thread, and optionally shows a skeleton loader — all for a list that's identical to the previous one.

### Solution
- `_displayStepsVersion` (int): incremented whenever the step **structure** changes.
- `_lastRebuildVersion` (int): set to `_displayStepsVersion` after each completed rebuild.
- `RefreshDisplaySteps()` returns early if `_displayStepsVersion == _lastRebuildVersion` (and not `immediate`).
- Version is bumped by `BumpDisplayVersion()` which is called from:
  - `OnMacroStepsCollectionChanged` — subscribed to `MacroSteps.CollectionChanged` in both constructors
  - `IsDelayHidden` setter — toggling delay visibility changes what's displayed
  - `IsSmartMode` setter — switching smart/raw mode changes the display

### Key Code
```
MacroEditorViewModel.Properties.cs:
- _displayStepsVersion (field)
- _lastRebuildVersion (field)
- BumpDisplayVersion()
- HookMacroCollection(MacroItem) — subscribes to CollectionChanged
- OnMacroStepsCollectionChanged — calls BumpDisplayVersion()
- RefreshDisplaySteps: early return when versions match
- RefreshAllMetadata: sets _lastRebuildVersion after rebuild completes
```

### What Could Go Wrong
- **Stale timeline**: If a structural change doesn't bump the version (e.g. a new CollectionChanged source we missed), the timeline could show old data. Symptom: add/remove step but timeline doesn't update.
- **Fix**: Call `ForceRefreshDisplaySteps()` (bypasses version check) or `ForceRefreshTimeline()` (synchronous, full rebuild).
- **Edge case**: Steps added/removed inside `ChildSteps` or `ChildStepsFalse` — currently only the top-level `MacroSteps` collection is subscribed. If nested collection changes don't trigger a top-level `CollectionChanged` event, the version won't bump. In practice, nested changes go through code paths that call `ForceRefreshDisplaySteps()` explicitly, so this is safe.

---

## Fix #3 — StepsEqual Fast Reject

### Problem
`UndoRedoService.StepsEqual()` recurses into child steps only after comparing all ~40 fields. For steps with children, the recursion overhead adds up.

### Solution
- Before the `for` loop over `ChildSteps`, compare `childA[0]` vs `childB[0]` first. If the first child differs, return false immediately without entering the loop.
- Same for `ChildStepsFalse`.

### What Could Go Wrong
- **False positive equality**: If `childA[0] == childB[0]` but `childA[1] != childB[1]`, the method still correctly enters the loop and finds the difference at index 1. The fast reject only short-circuits when the first child is different — it never skips a real difference.
- **No risk here.** This is a pure optimization with no behavioral change.

---

## Fix #4 — MoveSelectedSteps O(N²) Eliminated

### Problem
`MoveSelectedSteps()` (called by Move Up/Down keyboard shortcut and context menu) had two O(N²) patterns:

1. **`GroupBy(s => FindParentCollection(...))`**: `FindParentCollection` is a recursive tree walk (O(N) on total steps). Called once per selected step → O(N × N).
2. **`OrderBy(s => collection.IndexOf(s))` + loop `IndexOf`**: `IndexOf` on `ObservableCollection` is O(N). Called twice per step (sort + move) → O(N²).

For 50 selected steps in a 200-step macro: 50 × 200 = 10,000 operations just for grouping, plus 50 × 200 = 10,000 for index lookups.

### Solution
1. **Parent map**: `BuildParentMap()` does a single recursive pass, building `Dictionary<MacroStep, ObservableCollection<MacroStep>>`. GroupBy then uses dictionary lookup (O(1)) instead of `FindParentCollection` (O(N)).
2. **Index map**: Before processing each group, build `Dictionary<MacroStep, int>` from the collection. `OrderBy` and loop both use dictionary lookup (O(1)) instead of `IndexOf` (O(N)). Index map is updated incrementally after each `Move()` call (only 2 entries change per move).

### Key Code
```
MacroEditorViewModel.Core.cs:
- BuildParentMap(ObservableCollection, Dictionary, parent) — recursive, single pass
- MoveSelectedSteps: builds parentMap before GroupBy
- Index map built per collection before sort + move loop
- Index map updated after each collection.Move()
```

### What Could Go Wrong
- **Missing nested steps in parent map**: `BuildParentMap` recurses into `ChildSteps` and `ChildStepsFalse`. If a step exists in a nested collection but isn't reachable via recursion (e.g. added dynamically after map is built), the parent map won't contain it. Symptom: step won't move. **Fix**: The map is rebuilt on every `MoveSelectedSteps` call, so dynamic additions between calls are fine.
- **Index map drift**: After `collection.Move(idx, idx+1)`, only `indexMap[real]` and `indexMap[displaced]` are updated. If `Move()` fires `CollectionChanged` which modifies the collection in unexpected ways, the map could drift. **Unlikely** — `ObservableCollection.Move` is atomic and fires exactly one event.
- **VirtualSourceSteps handling**: Steps with `VirtualSourceSteps` (Smart View virtual blocks) are resolved to their raw steps before lookup. If `VirtualSourceSteps` is stale, the parent map lookup could fail. **Symptom**: Smart View block doesn't move. **Fix**: The code falls back to `CurrentMacro.MacroSteps` if the lookup fails.

---

## Fix #5 — Single-Pass Metadata Refresh

### Problem
`ExecuteGlobalStepChanged()` fires on **every step property change** (every keystroke in a text field, every checkbox toggle). It called 4 methods:

1. `UpdateErrorCount()` — full recursive traversal counting `!s.IsValid`
2. `RefreshAvailableNamedBlocks()` — full recursive traversal collecting named block names
3. `RefreshAvailableVariableNames()` — full recursive traversal collecting variable names
4. `ValidateDuplicateNames()` — LINQ `GroupBy` + `FirstOrDefault` per step

That's 4 full O(N) traversals + LINQ allocations on every property edit. For a 1000-step macro, that's 4000 node visits per keystroke.

### Solution
New `RefreshAllMetadata()` does a **single** `TraverseAllSteps` pass with a visitor that collects all 3 categories simultaneously:

```csharp
TraverseAllSteps(CurrentMacro.MacroSteps, s =>
{
    if (!s.IsValid) errorCount++;
    if (/* is named block */) namedBlockSet.Add(s.StepName);
    if (/* is user input */) variableNameSet.Add(s.InputVariableName);
});
```

Then duplicate names are computed from a `Dictionary<string, int>` count pass on top-level steps only.

Individual methods (`UpdateErrorCount`, `RefreshAvailableNamedBlocks`, `RefreshAvailableVariableNames`) are preserved for callers outside the hot path (e.g. `ForceRefreshDisplaySteps`, recording stop, capture completion).

### Key Code
```
MacroEditorViewModel.Properties.cs:
- RefreshAllMetadata() — single-pass, replaces 4 calls in ExecuteGlobalStepChanged
- SyncCollection(ObservableCollection<string>, HashSet<string>) — incremental diff update
- ExecuteGlobalStepChanged() — now calls only RefreshAllMetadata()
```

### Callers Updated
- `Core.cs` AddStep (~line 409): replaced 3-call pattern with `RefreshAllMetadata()`
- `Core.cs` DeleteStep (~line 471): replaced 3-call pattern with `RefreshAllMetadata()`

### What Could Go Wrong
- **Missing named blocks / variable names**: If the visitor condition is wrong (e.g. wrong `MacroStepType` check), the dropdowns could be empty or contain stale values. **Symptom**: Can't select a named block in Image Search config, or Text block dropdown is empty.
- **Error count mismatch**: If `s.IsValid` logic changed but `RefreshAllMetadata` wasn't updated, the error count could be wrong. **Symptom**: Save button grayed out when it shouldn't be (or vice versa).
- **SyncCollection diff issues**: The `SyncCollection` helper removes items not in the source, then adds new items. If `ObservableCollection` fires events during this diff, it could cause re-entrancy. **Mitigation**: The diff is simple and non-recursive, so re-entrancy is extremely unlikely.
- **Top-level-only duplicate check**: Duplicate names are only checked on `CurrentMacro.MacroSteps` (top level), not nested child steps. This is intentional — child steps don't appear in name dropdowns. If a future feature adds child step names to dropdowns, this logic needs updating.

---

## Fix #6 — ValidateDuplicateNames O(N) (merged into Fix #5)

### Problem
`ValidateDuplicateNames()` used LINQ `GroupBy(s => s.StepName)` then for each step called `FirstOrDefault(g => g.Key == step.StepName)?.Count()`. The `FirstOrDefault` is a linear scan through groups, making it O(K²) where K is the number of named steps.

### Solution
Replaced with `Dictionary<string, int>` counting:

```csharp
var nameCounts = new Dictionary<string, int>();
foreach (var s in CurrentMacro.MacroSteps)
{
    if (s.HasCustomName && !string.IsNullOrWhiteSpace(s.StepName))
    {
        nameCounts.TryGetValue(s.StepName, out var c);
        nameCounts[s.StepName] = c + 1;
    }
}
```

Then a single pass sets `IsDuplicateName`:

```csharp
foreach (var s in CurrentMacro.MacroSteps)
{
    s.IsDuplicateName = s.HasCustomName && nameCounts.TryGetValue(s.StepName, out var c) && c > 1;
}
```

### What Could Go Wrong
- **`IsDuplicateName` setter fires `GlobalStepChanged`**: The setter has a `OnPropertyChanged` which fires `GlobalStepChanged`. But `ExecuteGlobalStepChanged` is the method that calls `RefreshAllMetadata()`, and inside `RefreshAllMetadata`, the duplicate name check uses a dictionary (not the property setters). **No re-entrancy risk** — the dictionary is populated independently, then properties are set. The setters fire events, but they happen after the dictionary is built.

---

## Data Flow After All Fixes

```
User edits a step property
  → MacroStep.OnPropertyChanged fires
  → GlobalStepChanged event fires
  → OnGlobalStepChanged() (MacroEditorViewModel)
  → ExecuteGlobalStepChanged()
    → IsDirty = true
    → RefreshAllMetadata()  ← single O(N) pass
      → ErrorCount updated
      → AvailableNamedBlocks synced
      → AvailableVariableNames synced
      → IsDuplicateName flags set via Dictionary
    → IsSaveReady / CanPreview notified

User presses Move Up/Down
  → MoveSelectedSteps()
    → BuildParentMap() ← single O(N) pass
    → GroupBy parent map ← O(1) per step
    → Per group: build index map ← O(N) per collection
    → Sort + move ← O(1) per operation
    → Collection.Move fires CollectionChanged
    → BumpDisplayVersion()
    → RefreshDisplaySteps() checks version
      → If version changed: rebuild via Task.Run
      → If version unchanged: skip (return early)
```

---

## UndoRedoService — Signature System

The signature system (Fix #1) adds parallel stacks to the undo/redo mechanism:

```
_undoStack:     LinkedList<List<MacroStep>>  (existing)
_undoSignatures: LinkedList<ulong>           (new — parallel)

_redoStack:     LinkedList<List<MacroStep>>  (existing)
_redoSignatures: LinkedList<ulong>           (new — parallel)
```

Every time a snapshot is pushed to `_undoStack`, its signature is pushed to `_undoSignatures`. Trim/clear operations mirror both stacks.

### Signature Computation
`ComputeSignature()` recursively hashes:
- Step count
- For each step: Id, Type, StepName, Value, child counts

It does **not** hash every field — just the identity fields. Two steps that differ only in `IsDisabled` or `HumanizationLevel` will have the **same** signature. This is intentional — we want to detect "user toggled a checkbox and toggled it back" as "no change".

### Risk: False Signature Match
If two genuinely different states happen to produce the same signature (e.g. step A changed Type from X to Y, and step B changed Type from Y to X in a way that cancels out the hash), the clone could be skipped incorrectly. This is extremely unlikely given the hash includes Id + Type + StepName + Value, but worth knowing about.

---

## How to Debug Future Issues

### Symptom: Stale timeline / steps not appearing
1. Check `_displayStepsVersion` vs `_lastRebuildVersion` (add DebugLogger call)
2. Check if `ForceRefreshDisplaySteps()` is being called (bypasses version check)
3. Check if the collection change went through `MacroSteps.CollectionChanged`

### Symptom: Undo/Redo not working / missing states
1. Check `DebugLogger` for `VALUE_CHANGE skipped (reverted)` — is it skipping when it shouldn't?
2. Check `_pendingValueChangeSignature` vs current signature
3. Check if `StopWatching` is being called during undo/redo flow (it clears the snapshot)

### Symptom: Move Up/Down moves wrong steps
1. Check if parent map is correct — add DebugLogger in `BuildParentMap`
2. Check if `VirtualSourceSteps` resolution is correct
3. Check if index map is drifting after `Move()` calls

### Symptom: Wrong error count / empty dropdowns
1. Check `RefreshAllMetadata()` visitor conditions
2. Check if `SyncCollection` is diffing correctly
3. Check if `IsDuplicateName` setter is firing `GlobalStepChanged` (re-entrancy)

---

## Related Files Quick Reference

| File | What Changed |
|------|-------------|
| `UndoRedoService.cs` | Signature system, clone skip, StepsEqual fast reject |
| `MacroEditorViewModel.Properties.cs` | `_suppressDirtyCheck`, `RefreshAllMetadata()`, `SyncCollection()`, display rebuild skip, version tracking |
| `MacroEditorViewModel.Core.cs` | `BuildParentMap()`, `MoveSelectedSteps` rewrite, AddStep/DeleteStep → `RefreshAllMetadata()` |
| `MacroEditorViewModel.cs` | `HookMacroCollection()` in constructors |

---

## Reverting These Changes

If you need to revert all performance optimizations and go back to the original behavior:

1. **UndoRedoService.cs**: Remove `_pendingValueChangeSignature`, `_lastCommittedSignature`, `_undoSignatures`, `_redoSignatures`, and all signature-related checks in `PushState`, `Tick`, `FlushPendingChanges`, `StopWatching`, `Undo`, `Redo`, `Clear`.
2. **MacroEditorViewModel.Properties.cs**: Remove `_displayStepsVersion`, `_lastRebuildVersion`, `BumpDisplayVersion`, `HookMacroCollection`, `OnMacroStepsCollectionChanged`, the early return in `RefreshDisplaySteps`, and `RefreshAllMetadata`. Restore the 4 separate calls in `ExecuteGlobalStepChanged`.
3. **MacroEditorViewModel.Core.cs**: Remove `BuildParentMap`. Restore `FindParentCollection` in GroupBy and `collection.IndexOf` in MoveSelectedSteps. Restore 3-call pattern in AddStep/DeleteStep.

**Important:** Reverting Fix #2 (display rebuild skip) alone is safe — just remove the early return check. The version tracking has no side effects when not checked.
