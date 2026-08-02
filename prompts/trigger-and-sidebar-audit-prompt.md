# Trigger Modes & Sidebar Audit Prompt

Copy-paste this entire prompt into a new session.

---

## Your Job

You are a QA auditor for **PowerX Keys** — a WPF C# macro automation app. Your task has **two phases**:

### Phase 1: Scan all Trigger Modes
### Phase 2: Scan the Sidebar / Right Panel (macro list, hotkey badges, profiles, etc.)

For each phase:
1. Read the relevant code
2. Write down ALL findings (bugs, edge cases, missing validations, UI issues)
3. **STOP and ask permission** before fixing anything

---

## Rules

- Scan one trigger mode at a time, then one sidebar feature at a time
- Write ALL findings to an audit file FIRST
- Do NOT fix anything until the user says "go" or "do it"
- Keep findings clear and short — use the format below

---

## Phase 1: Trigger Modes

The app supports these trigger modes (enum `TriggerMode` in `Models/AppConfig.cs`):

1. **Single** — Press hotkey once to fire
2. **PressAndRelease** — Fire on key release
3. **DoubleTap** — Double-press hotkey to fire
4. **Hold** — Hold key to fire (with configurable duration)
5. **Release** — Fire on release
6. **LongPress** — Hold for X ms then fire
7. **Toggle** — First press starts, second press stops
8. **ScreenEvent** — Auto-fire when image/pixel found on screen
9. **Schedule** — Fire on a timed interval
10. **MobileRemote** — Fire from mobile app via HTTP

### What to check per trigger mode:
- **Compilation:** How it's compiled into AHK in `ScriptCompilerService.cs` — any missing cases? Wrong syntax?
- **UI:** How the user selects/configures it in `CustomActionCard.xaml` — any broken bindings? Hidden options?
- **Validation:** What happens with invalid config? (no hotkey, zero duration, etc.)
- **Edge cases:** Toggle stuck? Double-tap timing issues? Schedule running after disable?
- **Live reload:** Does changing trigger mode while engine is running update correctly?

---

## Phase 2: Sidebar / Right Panel

The sidebar is `Views/ScriptLibraryView.xaml` with ViewModel `ViewModels/ScriptLibraryViewModel.*.cs`

### What to check:
- **Macro list display** — sorting, filtering, grouping, empty states
- **Hotkey badges** — correct display of key combos, trigger mode indicators
- **Enable/Disable toggle** — does it properly start/stop individual macros?
- **Profile system** — switching profiles, creating, deleting, what happens to active macros
- **Search/filter** — does it work with special characters, empty results?
- **Drag-drop / reorder** — if supported, any edge cases?
- **Right-click context menu** — all options working?
- **Conflict detection** — duplicate hotkeys, overlapping triggers
- **New macro creation flow** — from sidebar to editor and back
- **Delete macro** — cleanup (files, references, engine reload)

---

## Output Format

Write findings to: `c:\Users\Maaz\Documents\New folder\Obsidian Vault\core\wiki\audits\trigger-sidebar-audit.md`

Use this format per finding:
```
### [SEVERITY: Critical/Medium/Low] — Short title
**Phase:** Trigger Modes / Sidebar
**Component:** (e.g., Toggle mode, Profile system)
**Scenario:** What triggers it
**Impact:** What goes wrong
**Verified:** Yes/No
```

At the end, add a summary table of all findings.

---

## Project Location

```
c:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\PowerX_Keys_V2\
```

## Key Files

| What | Path |
|------|------|
| TriggerMode enum | `Models/AppConfig.cs` (search `enum TriggerMode`) |
| ActionItem model | `Models/AppConfig.cs` (search `class ActionItem`) |
| Script compilation | `Services/ScriptCompilerService.cs` |
| Sidebar view | `Views/ScriptLibraryView.xaml` |
| Sidebar ViewModel | `ViewModels/ScriptLibraryViewModel.*.cs` (multiple partial files) |
| Macro card (per-item) | `Views/CustomActionCard.xaml` + `.cs` |
| Trigger mode converter | `Converters/TriggerModeToReadableConverter.cs` |
| Profile logic | search for `Profile` in ViewModel files |

## Read First

- `c:\Users\Maaz\Documents\New folder\Obsidian Vault\core\SOUL.md`
- `c:\Users\Maaz\Documents\New folder\Obsidian Vault\core\SCHEMA.md`
- `c:\Users\Maaz\Documents\New folder\Obsidian Vault\core\GOTCHAS.md`

---

## Communication Rules

- English only
- Short findings, bullet points
- Don't explain what you're about to do — just do it
- Keep outputs minimal
- **STOP after writing the audit** — ask for permission before fixing

---

## Start

Begin with Phase 1, Trigger Mode #1 (Single). Read the compilation code, UI, validation. Write findings. Move to the next mode. After all 10 modes, move to Phase 2.
