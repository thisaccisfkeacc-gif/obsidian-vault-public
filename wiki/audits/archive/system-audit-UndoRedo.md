# System Audit: Undo/Redo (Deep)

**Date:** 2026-07-24
**Scope:** UndoRedoService, undo/redo stack management, property tracking

---

## BUGS

### 1. DispatcherTimer.Tick memory leak
- **Severity:** Medium
- **Location:** UndoRedoService.cs:175-199
- **Problem:** Tick handler never unsubscribed in `StopWatching()`. Leaked timer delegates accumulate over repeated StartWatching/StopWatching cycles.

### 2. StepsEqual missing 20+ properties
- **Severity:** Medium
- **Location:** UndoRedoService.cs:120-127
- **Problem:** Only compares Type, Value, Duration, X, Y, IsDisabled, StepName, ClickCount, WindowTitle. Missing: ActionTarget, KeyActionType, ScrollAmount, EndX, EndY, Tolerance, LogicMode, LogicSource, UIAction, UIFindMode, InputVariableName, InputType, WaitConditionType, OnTimeoutAction, and more. Duplicate guard incorrectly skips push when meaningful changes occurred.

### 3. `_lastCommittedSnapshot` race
- **Severity:** Medium
- **Location:** UndoRedoService.cs:99
- **Problem:** Written outside `_stackLock`. Debounce timer and CommitRestPoint can race, causing snapshot to point at stale state.

### 4. CollectionChanged + debounce ghost entries
- **Severity:** Medium
- **Location:** UndoRedoService.cs:211-231
- **Problem:** Structural and value changes in same debounce window can push wrong before-state, corrupting undo history.

### 5. IsDisabled cascading consumes 2 undo slots
- **Severity:** Low
- **Location:** MacroEditorViewModel.Commands.cs:1455
- **Problem:** Single disable toggle creates duplicate undo entry via cascade + debounce.

### 6. Clone doesn't preserve IsSynthetic
- **Severity:** Low
- **Location:** MacroItem.cs Clone method
- **Problem:** Undo snapshots lose `IsSynthetic` state. Smart View grouping may change after undo.

### 7. RemoveStep with BeginInvoke
- **Severity:** Low
- **Location:** MacroEditorViewModel.Core.cs:381-388
- **Problem:** Undo during queued background removal corrupts restored state.

---

## DEAD CODE

1. `CommitRestPoint` — Never called from any ViewModel code
2. `_suppressUndoPush`, `_suppressRefresh`, `_bulkDeleteProtectedSteps` — Could be local variables

---

## VERIFIED OK

- Deep clone via `Clone(true)` — properly recursive
- Stack bounding — MaxLevels 5-100, default 50
- Redo stack clearing — correct on new mutations
- `_isRestoring` flag — prevents re-push during Undo/Redo
- Property change debouncing — 500ms reasonable
- `_trackedProperties` completeness — covers ~90+ properties
- Nested child subscription — correct with duplicate prevention
- Recording integration — single Ctrl+Z undoes entire session
- File cleanup on undo — correctly deferred
