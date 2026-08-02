---
tags: [feature, service, architecture]
date: 2026-07-15
sources:
  - PowerX_Keys_V2/Services/UndoRedoService.cs
  - PowerX_Keys_V2/ViewModels/MacroEditorViewModel.Commands.cs
status: active
---

# Undo/Redo Edge Case Audit

**Summary:** Deep audit of `UndoRedoService.cs` for edge cases that could break undo/redo behavior.

---

### [SEVERITY: Critical] — Mixed operation loses pending value change

**Scenario:** User edits a property (e.g., changes Duration), then within 500ms adds a new block. `PushState()` is called by the add-block code, which snapshots the *current* state (already containing the edit). The debounce timer fires later but `SnapshotsEqual` sees the snapshot matches what was just pushed, so it's silently skipped. The "before-edit" state stored in `_lastCommittedSnapshot` is never pushed to the undo stack.

**Impact:** The property change is baked into the add-block undo entry. Undoing restores pre-add AND pre-edit state, but the user perceives them as two distinct actions. If they only wanted to undo the add, the value change is silently reverted too — invisible data loss.

**Fix:** At the top of `PushState()`, call `FlushPendingChanges()` (or inline the same logic) so any buffered value-change snapshot is committed *before* the structural snapshot is pushed. This guarantees two separate undo entries.

---

### [SEVERITY: Critical] — `SnapshotsEqual` ignores nested children entirely

**Scenario:** User edits a property on a child step inside an If/Else or Loop container. The top-level step list hasn't changed (same count, same types, same top-level values). `SnapshotsEqual` returns `true`, so the snapshot is rejected as a duplicate.

**Impact:** Undo entries for nested-property edits can be silently dropped if the parent step's top-level fields happen to match. The user makes a change, presses Ctrl+Z, and nothing happens.

**Fix:** Recurse into `ChildSteps` and `ChildStepsFalse` inside `SnapshotsEqual`. Compare child count and key fields at each level. Alternatively, add a simple hash/version counter to each step that increments on any tracked property change — then compare that instead.

---

### [SEVERITY: Medium] — `_lastCommittedSnapshot` not updated after `PushState()`

**Scenario:** A structural operation calls `PushState()`, which sets `_lastCommittedSnapshot = snapshot` (the snapshot of *before* the mutation). But the mutation then happens (e.g., step added). Now `_lastCommittedSnapshot` reflects the state *before* the structural change, not *after* it. If the user edits a value next, the debounce fires and pushes `_lastCommittedSnapshot` — which is the pre-add state, not the post-add state.

**Impact:** An extra ghost undo entry appears that reverts the structural add. Pressing undo twice undoes the value edit AND the add, even though only one undo was expected.

**Fix:** After any `PushState()` call that is followed by a mutation, update `_lastCommittedSnapshot` to reflect the post-mutation state. Best approach: have `PushState()` NOT set `_lastCommittedSnapshot`; instead, let `StartWatching()` (which is always called after undo/redo restore) capture it. For non-undo structural operations, add an explicit `_lastCommittedSnapshot = DeepCloneSteps(currentSteps)` after the mutation completes.

---

### [SEVERITY: Medium] — Undo/Redo doesn't update `_lastCommittedSnapshot`

**Scenario:** User undoes, then edits a value. After undo, `StartWatching()` is called which sets `_lastCommittedSnapshot` correctly. But if `StartWatching()` is called with a null or empty collection (edge case where `CurrentMacro?.MacroSteps` is null), `_lastCommittedSnapshot` stays stale from before the undo.

**Impact:** Next value change pushes a stale/wrong "before" snapshot. Undo produces unexpected state.

**Fix:** Add a null guard in `StartWatching` — if steps is null, also null out `_lastCommittedSnapshot` to prevent stale usage. The debounce tick already checks for null, so this is safe.

---

### [SEVERITY: Medium] — `FlushPendingChanges` clears redo without a new user action

**Scenario:** User undoes, then the system calls `FlushPendingChanges()` at the top of the Undo command handler. If `_pendingValueChangePush` is true (from a prior edit), flush pushes the old snapshot and calls `_redoStack.Clear()` — destroying the redo stack that was just populated by the previous undo.

**Impact:** User presses Undo twice quickly. First undo works, but if the flush from the first undo clears redo, the state from the first undo is gone from redo.

**Fix:** In the Undo/Redo command handlers, the flush is already called before the undo operation. But `FlushPendingChanges` should NOT clear `_redoStack` when called from within an undo/redo context. Add a parameter `bool preserveRedo = false` or check `_isRestoring` (though it's not set yet at flush time). Alternatively, move the flush logic to only clear redo when it's a genuine new user edit, not a pre-undo housekeeping call.

---

### [SEVERITY: Medium] — Debounce timer holds reference to dead collection after StopWatching

**Scenario:** `StopWatching()` nulls `_watchedSteps` and stops the timer, but the lambda captured by the timer's Tick event still references `_watchedSteps` (via closure). If `StopWatching` is called while `_pendingValueChangePush` is true, the pending change is silently lost — never pushed.

**Impact:** If the user makes an edit, then immediately switches macros (triggering StopWatching), the last edit is unrecoverable.

**Fix:** Call `FlushPendingChanges()` at the top of `StopWatching()` before nulling anything. This ensures any buffered edit is committed.

---

### [SEVERITY: Low] — Rapid edits: debounce works correctly (no bug)

**Scenario:** User types several characters quickly in a text field, each firing PropertyChanged. The debounce timer restarts on each event.

**Impact:** None — this works as designed. Only one undo entry is created after 500ms of inactivity, capturing the "before" state. Verified by reading the timer reset logic in `OnStepPropertyChanged`.

**Fix:** None needed.

---

### [SEVERITY: Low] — Undo-then-edit correctly branches (no bug)

**Scenario:** User undoes, then makes a new edit. `PushState()` or the debounce flush both call `_redoStack.Clear()`.

**Impact:** None — redo is correctly invalidated on new edits. The new branch starts fresh.

**Fix:** None needed.

---

### [SEVERITY: Low] — No deadlock risk in current architecture

**Scenario:** `_stackLock` is used inside both `PushState` (UI thread via debounce) and `Undo`/`Redo` (UI thread via command). The DispatcherTimer fires on the UI thread, same as commands.

**Impact:** Since everything runs on the same UI thread, the lock is effectively single-threaded — no contention. The lock exists as a safety net for hypothetical background access but won't deadlock in current usage.

**Fix:** None needed. The lock is defensive and harmless.

---

### [SEVERITY: Low] — Double-subscribe prevention works but is fragile

**Scenario:** `SubscribeStep` does `step.PropertyChanged -= OnStepPropertyChanged` before `+=`. This correctly prevents double-subscribe. `SubscribeChildCollection` uses a dictionary to track handlers.

**Impact:** Currently safe. However, if a step instance moves between collections (e.g., drag from one If-branch to another), `UnsubscribeStep` is called on remove and `SubscribeStep` on add — correct lifecycle.

**Fix:** None urgently needed. Consider adding a `HashSet<MacroStep>` of currently-subscribed steps for O(1) "am I already watching this?" checks, as the `-= then +=` pattern silently succeeds even if the step was never subscribed.

---

### [SEVERITY: Low] — `_lastCommittedSnapshot` initial state edge case

**Scenario:** If `StartWatching` is called with an empty collection, `_lastCommittedSnapshot` is an empty list. If the user then adds a step (which calls `PushState`), the empty snapshot is pushed correctly. But if `PushState` also sets `_lastCommittedSnapshot` to the same empty snapshot, the next debounce won't detect it's stale.

**Impact:** Minimal in practice — the empty-to-one-step transition is handled by `PushState` directly.

**Fix:** Low priority. Ensure `PushState` doesn't overwrite `_lastCommittedSnapshot` (addressed in the Critical finding above).

---

## Priority Summary

| # | Severity | Issue |
|---|----------|-------|
| 1 | Critical | Mixed operation loses pending value change |
| 2 | Critical | `SnapshotsEqual` ignores nested children |
| 3 | Medium   | `_lastCommittedSnapshot` stale after structural ops |
| 4 | Medium   | Undo/Redo + null MacroSteps leaves stale snapshot |
| 5 | Medium   | `FlushPendingChanges` clears redo during undo sequence |
| 6 | Medium   | `StopWatching` discards pending change silently |
| 7 | Low      | Debounce batching (works correctly) |
| 8 | Low      | Undo-then-edit branching (works correctly) |
| 9 | Low      | No deadlock risk |
| 10| Low      | Double-subscribe safe but fragile |

## Related Pages

- [[undo-redo-service]]
- [[macro-editor-commands]]

## Key Files

- [UndoRedoService.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Services/UndoRedoService.cs)
- [MacroEditorViewModel.Commands.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/ViewModels/MacroEditorViewModel.Commands.cs)
