---
purpose: Session Handoff — Active context for the next agent
project: PowerX Keys
date: 2026-08-04
---

# HANDOFF.md

---

## How Maaz Works
- Short, simple, friendly — no jargon
- Ask before adding extras
- Don't run build/run unless asked (or told to auto-build at end)
- Read files before editing
- After fixing, just confirm — no recap unless asked

---

## Current Work: "Auto Merge into Text" Context Menu Feature (DONE — build verified)

### User Requirement (exact spec)
- Right-click a Keyboard step that is **followed by a Text step** (delays between are OK) and choose **"Auto Merge into Text"**.
- Merge result: one permanent Text step that combines the leading key + the text (e.g. `H` + `ello` → `Hello`).
- The leading Key step and any intermediate Delay steps are removed.
- Merge must persist to `CurrentMacro.MacroSteps` / SQLite.
- Raw Mode support unchanged. **No emoji, no lightning icon on the menu item.**

### What Was Done
1. **XAML** — `PowerX.UI/Views/MacroEditorView.xaml` (context menu area):
   - Added `MenuItem Header="Auto Merge into Text"` `Tag="AutoMergeIntoTextMenuItem"`.
   - `Command="{Binding PlacementTarget.Tag.AutoMergeTextCommand}"`, `CommandParameter="{Binding}"`.
   - Default `Collapsed`; `DataTrigger` shows it when the step is `Keyboard`.
2. **Code-behind** — `PowerX.UI/Views/MacroEditorView.Events.cs`:
   - In `OnTimelineContextMenuOpening`, look up the menu item by tag and toggle visibility via `vmAuto.CanAutoMergeIntoText(step)`.
3. **Command** — `PowerX.UI/ViewModels/MacroEditorViewModel.Commands.cs`:
   - Rewrote `AutoMergeTextCommand`:
     - Scans **forward** (Keyboard→Text) and **backward** (Text→Keyboard) for the pair.
     - Skips over Delay steps in between.
     - Resolves `VirtualSourceSteps` into real source steps.
     - Replaces the whole span with a single permanent `Text` step.
     - `UndoRedoManager.PushState` for undo, sets `IsDirty`, calls `RefreshDisplaySteps`.
4. **Helpers** — `PowerX.UI/ViewModels/MacroEditorViewModel.SmartView.cs`:
   - Added `CanAutoMergeIntoText`, `AddResolvedSources`, `ComputeLeadingChar` (handles single char and `SHIFT + H` chords).

### Verified
- `dotnet build` on `PowerX_Keys_V2.csproj` succeeded — 0 errors.

### Not Done Since Handoff
- None — feature is complete and built.

---

## In Progress: "Move Up" Mystery (UNRESOLVED — still investigating)

### User Report
- On the timeline, manually add: 1) a Mouse block, 2) a Keyboard block.
- Right-click the Keyboard block → **Move Up** → it does NOT move.
- **Hint given by user:** "I just assigned key and it works." → Move Up works after the Keyboard step has a key assigned (i.e. it fails only for a **blank / un-configured Keyboard step**).

### Investigation So Far (facts established)
- Call chain: `MoveStepUpCommand` → `MoveSelectedSteps(o, -1)` → `MoveStepCore(step, direction)` in `MacroEditorViewModel.Core.cs` (~line 699).
- `MoveStepCore` has two branches:
  - **Virtual branch**: when `step.VirtualSourceSteps.Count > 0`.
  - **Real branch**: `FindStepById` → `FindParentCollection` → `collection.IndexOf(realStep)` → `collection.Move(index, newIndex)`. Otherwise silently no-ops.
- Right-click target comes from `OnTimelineContextMenuOpening` (`MacroEditorView.Events.cs` ~line 945): `rightClickedBulkStep = contextMenu.DataContext as MacroStep`.
- `SetupBulkAwareness` single mode rebinds command + `CommandParameter = step`.
- `MacroStep.Id` defaults to `Guid.NewGuid()`.
- Batch-rule confusion: user said "just solve the mystery that I just assigned key and it works" — this is the KEY hint, not a second task.

### Leading Hypothesis (NOT confirmed)
- A blank Keyboard step in Smart View may become a **virtual/wrapper step** (or one whose backing real step isn't present in `CurrentMacro.MacroSteps`), so `FindStepById` / `collection.IndexOf` silently fails and no move happens.
- Once a key is assigned, the step becomes a real, findable step, and Move Up works.
- Alternative: the blank step is found, its span/bundle changes after `RefreshDisplaySteps`, making the move visually imperceptible.

### Next Steps (for next agent)
1. Confirm why a blank Keyboard step differs in the Smart View / `MoveStepCore` path vs. one with a key assigned.
   - Check `BuildSmartSteps` / `PopulateKeyboardBundling` in `MacroEditorViewModel.SmartView.cs` for how an un-valued Keyboard (`Value == null`, `KeyActionType = "Press"`) is emitted.
   - Check whether `OriginalStep` is set (pass-through) or whether it gets wrapped/absorbed.
   - Check `MapSnapshotToSteps` — when `OriginalStep != null` the SAME instance is re-added to the list; verify the right-clicked step is that same instance.
2. If it's a virtual-step issue: make `MoveStepCore`/`MoveSelectedSteps` operate on the resolved real target (via `VirtualSourceSteps`, or the real collection) so blank Keyboard steps can be moved.
3. Rebuild once and verify manually per user's repro (Mouse then Keyboard, right-click Keyboard, Move Up).

### Files Involved
| Topic | File |
|-------|------|
| Move logic | `MacroEditorViewModel.Core.cs` (`MoveStepCore`, `MoveSelectedSteps`, `FindStepById`, `FindParentCollection`) |
| Context menu wiring | `MacroEditorView.Events.cs` (`OnTimelineContextMenuOpening`, `SetupBulkAwareness`) |
| Smart View bundling | `MacroEditorViewModel.SmartView.cs` (`BuildSmartSteps`, `PopulateKeyboardBundling`, `MapSnapshotToSteps`) |
| Model | `MacroItem.cs` (`MacroSteps`, `VirtualSourceSteps`, `KeyActionType` default "Press", `Id`) |
| Menu items / Auto Merge | `MacroEditorView.xaml` |
| Auto Merge command / helpers | `MacroEditorViewModel.Commands.cs`, `MacroEditorViewModel.SmartView.cs` |

---

## Project Context
> See [[agent-onboarding]] for project layout, [[overview]] for architecture, and [[dual-execution-model]] for AHK vs C# execution.

---

## Where Detailed Work Lives
| Topic | File |
|-------|------|
| Earlier "Move Up looked dead" multi-select fix | `wiki/log.md` (~line 240) |
| Move to Top/Bottom removal | `wiki/log.md` (~line 272) |
| Smart Mode edge cases | `wiki/features/smart-mode-audit.md` |
| Smart Mode audit 3 (Move Up PASS note) | `audit_report_3.md` |

---

## Important Notes for Next Agent
- Do NOT edit `.ahk` scripts directly — always edit `ScriptCompilerService.cs`.
- Rebuild once after fixing something; auto-kill `PowerX Keys.exe` with `taskkill /f /im "PowerX Keys.exe"` if it blocks the build.
- If any other build error appears, STOP and tell the user — do not auto-fix unless it was your own mistake.
- Batch-task rule: if the user gives tasks one-by-one, explain each but do NOT fix until user says "go".
- Fix silently where possible; brief plain-English summary at the end.
- If not 100% sure a bug is real, do NOT touch it — tell the user in plain English instead.