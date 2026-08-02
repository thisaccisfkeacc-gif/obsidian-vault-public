---
tags: [prompt, fix-05]
date: 2026-07-23
status: ready
---

# Prompt: Fix 05 — Split God Classes

## Your Task

Investigate and implement splitting of 3 large god classes in the PowerX Keys codebase. Write your findings and implementation to `fixes/05-split-god-classes-ACTUAL.md`.

## Codebase Location

`C:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\PowerX_Keys_V2\`

## Files to Investigate

1. `ViewModels/ScriptViewModel.cs` (~1300 lines)
2. `ViewModels/ScriptEditorViewModel.cs` (~2800 lines)
3. `Services/ScriptCompilerService.cs` (~5264 lines)
4. `Services/MacroExecutionService.cs` (~2801 lines)

## What to Do

1. **Read each file** to understand its responsibilities
2. **Identify natural split points** — group related methods into logical units
3. **Use `partial class`** for splitting (keeps one class name, multiple files)
4. **Create the split files** in the same directory as the original
5. **Update the namespace/using statements** as needed
6. **Build and verify** — `dotnet build "c:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\PowerX_Keys_V2\PowerX_Keys_V2.csproj"`

## Splitting Strategy (from plan)

### ScriptCompilerService.cs (5264 lines) → 5 partial files:
- `ScriptCompilerService.cs` — Core engine & header emit
- `ScriptCompilerService.MouseClick.cs` — Mouse & coordinate blocks
- `ScriptCompilerService.ImagePixel.cs` — Image search & pixel detection
- `ScriptCompilerService.WindowLogic.cs` — Window management & scope control
- `ScriptCompilerService.Hotkeys.cs` — Hotkey registration & triggers

### MacroExecutionService.cs (2801 lines) → 4 partial files:
- `MacroExecutionService.cs` — Core execution
- `MacroExecutionService.Mouse.cs` — Mouse actions
- `MacroExecutionService.Keyboard.cs` — Keyboard actions
- `MacroExecutionService.ImagePixel.cs` — Image/pixel actions

### ScriptViewModel.cs (1300 lines) → Extract:
- `RecordingService.cs` — Recording logic
- `ScriptExecutionManager.cs` — Execution details

### ScriptEditorViewModel.cs (2800 lines) → Extract:
- `EditorPreviewManager.cs` — Preview logic
- `UndoRedoManager.cs` — Undo/redo
- `AiCodeGenerator.cs` — AI integration

## Rules

- **One extraction at a time** — build after each
- **No behavior changes** — just moving code
- **Use `partial class`** where possible
- **Update all `using` statements** if types move namespaces
- **Build must pass** — 0 errors

## Output

Write your results to: `C:\Users\Maaz\Documents\New folder\Obsidian Vault\App Optimization & Audit\fixes\05-split-god-classes-ACTUAL.md`

Include:
- What you split and why
- File names created
- Line counts before/after
- Any issues encountered
- Build result (pass/fail)
