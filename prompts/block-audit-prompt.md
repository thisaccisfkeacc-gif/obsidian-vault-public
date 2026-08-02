# Block-by-Block Audit & Fix Prompt

Copy-paste this entire prompt into a new session.

---

## Your Job

You are a QA auditor and bug fixer for **PowerX Keys** — a WPF macro automation app that compiles macros into AutoHotkey v2 scripts.

Your task: **Scan every block type one at a time**, find edge-case bugs, verify they're genuine, and fix them.

---

## Rules

- **One block at a time.** Do NOT scan multiple blocks in parallel.
- For each block, you will:
  1. Read how it's **compiled** (in `ScriptCompilerService.cs`)
  2. Read how it's **saved/loaded** (in `MacroItem.cs` / JSON serialization)
  3. Read how it's **displayed** (in `MacroStepCard.xaml` templates and inline templates)
  4. Read how it's **previewed** (in `MacroEditorViewModel.Commands.cs` → `UnifiedPreviewCommand`)
  5. Check if there are **validation issues** (missing values, null fields)
  6. Write findings to an audit file
  7. If a bug is genuine → fix it immediately
  8. Move to the next block

- **Write audit results** to: `c:\Users\Maaz\Documents\New folder\Obsidian Vault\core\wiki\audits\block-audit-[BlockName].md`
- Use this format per finding:
```
### [SEVERITY: Critical/Medium/Low] — Short title
**Scenario:** What triggers it
**Impact:** What goes wrong
**Verified:** Yes/No (did you confirm in the code?)
**Fixed:** Yes/No
**Fix details:** (if fixed)
```

- If a block has **zero issues**, still create the file and write "✓ No issues found"
- After ALL blocks are done, create a summary file: `block-audit-summary.md`

---

## Project Location

```
c:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\PowerX_Keys_V2\
```

## Key Files to Read

| What | Path |
|------|------|
| Block types enum | `Models/MacroItem.cs` (top of file) |
| Block model (all properties) | `Models/MacroItem.cs` (MacroStep class) |
| Script compilation | `Services/ScriptCompilerService.cs` |
| Preview logic | `ViewModels/MacroEditorViewModel.Commands.cs` (search `UnifiedPreviewCommand`) |
| Block card templates | `Views/Templates/KeyboardInputTemplates.xaml` and `Views/MacroStepCard.xaml` |
| Smart View bundling | `ViewModels/MacroEditorViewModel.SmartView.cs` |
| Step deletion | `ViewModels/MacroEditorViewModel.Core.cs` (search `RemoveStep`) |
| Adding new steps | `ViewModels/MacroEditorViewModel.Core.cs` (search `AddNewStep`) |

## Read First (Project Context)

- `c:\Users\Maaz\Documents\New folder\Obsidian Vault\core\SOUL.md`
- `c:\Users\Maaz\Documents\New folder\Obsidian Vault\core\SCHEMA.md`
- `c:\Users\Maaz\Documents\New folder\Obsidian Vault\core\GOTCHAS.md`

---

## Block Types (Process in this order)

1. **Delay** — Timed pause (Duration ms)
2. **Keyboard** — Key press/hold/release
3. **MouseClick** — Click, drag, scroll, hold/release
4. **MouseTrace** — Recorded mouse path replay
5. **Text** — Type text block
6. **ImageSearch** — Find image on screen
7. **PixelSearch** — Find pixel color on screen
8. **WindowAction** — Maximize, minimize, close, move window
9. **UIElement** — UI Automation (click button, read text, etc.)
10. **LogicIf** — If/Else conditional branching
11. **LoopSequence** — Repeat N times
12. **GroupHeader** — Folder grouping (no execution)
13. **Popup** — Show dialog message
14. **Notification** — Tray notification
15. **SystemSound** — Play audio alert
16. **UserInput** — Ask user for input during execution
17. **WaitForKey** — Pause until key press
18. **WaitUntil** — Wait until condition met (image/pixel/timeout)
19. **FileLauncher** — Open file/app/URL
20. **CallMacro** — Execute another macro
21. **SetVariable** — Set a runtime variable

---

## What to Check Per Block

- **Compilation:** Does the AHK output handle all property combinations? Null values? Empty strings?
- **Save/Load:** Are all properties serialized? Anything lost on round-trip?
- **Display:** Does the card show correct info? Binding issues?
- **Preview:** Does it work? Does it crash on missing values?
- **Validation:** Does `IsValid` catch all genuinely invalid states?
- **Edge cases:** What happens with extreme values? (Duration=0, empty text, no coordinates, etc.)

---

## Communication Rules

- English only
- Short findings, bullet points
- Don't explain what you're about to do — just do it
- After fixing, confirm it's done — no recap unless asked
- Keep outputs minimal

---

## Start

Begin with block #1 (Delay). Read the relevant code, audit it, write findings, fix if needed, then move to #2.
