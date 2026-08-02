---
tags: [guide, onboarding, agent, architecture]
date: 2026-08-01
sources:
  - ViewModels/
  - Models/
  - Managers/
  - Services/
status: current
---

# Agent Onboarding — Read This First 🚀

> **If you're a new AI agent starting work on PowerX Keys, read this entire page before touching any code.**

> [!tip] **Also read [[index]]** — It is the master catalog of all wiki pages.

## What Is PowerX Keys?

PowerX Keys is a **desktop macro automation app** for Windows. Users record or build macros (mouse movements, key presses, image searches, pixel detection) and bind them to hotkeys. Under the hood, the app generates AutoHotkey v2 scripts and runs them via a white-labeled engine.

- **Tech Stack:** C# .NET 10.0, WPF + WinForms hybrid, AutoHotkey v2, SQLite
- **Pattern:** MVVM (Model-View-ViewModel)
- **Version:** comes from the app project version in `PowerX_Keys_V2.csproj`; UI reads it through `VersionInfo.cs`

## Project Layout

The solution has **4 projects** (plus the updater). Code lives in the three libraries; the main app project only hosts startup/bootstrapping:

```
PowerX_Keys_V2_Rebuild/
├── PowerX_Keys_V2/          ← Main app (startup, App.xaml.cs, MainWindow)
├── PowerX.Core/             ← Models (MacroItem, AppConfig, AppEnums, AppConstants, etc.)
├── PowerX.Services/         ← Services/ + Managers/ (ScriptCompilerService, ScriptManager, MacroDatabase, ConfigManager, etc.)
├── PowerX.UI/               ← ViewModels/ + Views/ (WPF XAML + code-behind)
├── PowerX_Updater/          ← Separate updater app
└── _Archive/                ← Archived branches, legacy code
```

## The Golden Rule: How Macros Work

This is the **single most important concept** to understand:

```
JSON Config → ScriptCompilerService → .ahk script → PowerX_Engine.exe
```

1. User builds a macro in the editor (a list of `MacroStep` objects)
2. Macro is saved to **SQLite** via `MacroDatabase`
3. Hotkey bindings are saved to **JSON** via `ConfigManager`
4. `ScriptCompilerService.CompileMasterScript()` reads both sources and generates an `.ahk` file
5. `ScriptManager.Start()` launches `PowerX_Engine.exe` (a renamed AutoHotkey64.exe) to run the script

**Two execution paths exist:**
- **AHK scripts** (primary) — handles hotkeys, mouse moves, image/pixel search
- **C# P/Invoke** (secondary) — `SmoothTraceEngine` uses `SetCursorPos` for recorded mouse paths

## Key Singletons & Entry Points

| Class | Purpose |
|-------|---------|
| `ConfigManager` | Loads/saves JSON config (`config.json`). Access via `ConfigManager.Current` |
| `MacroDatabase` | SQLite CRUD for macros. Static methods like `LoadAllMacros()`, `SaveMacro()` |
| `ScriptManager` | Manages AHK process lifecycle. 6 process slots: master, executor, macro, tester, recorder, snippets |
| `ScriptCompilerService` | Generates AHK scripts from JSON + SQLite data |
| `MainViewModel` | Root VM. Controls navigation between pages |

## Data Storage

| What | Where | Format |
|------|-------|--------|
| Macros | `%LOCALAPPDATA%/PowerXKeys/Configs/macros.db` | SQLite (each step is an individual row + `ExtraJson`) |
| Settings & Hotkeys | `%LOCALAPPDATA%/PowerXKeys/config.json` | JSON |
| Captured images | `%LOCALAPPDATA%/PowerXKeys/Engine/Images/` | PNG |
| Mouse traces | `%APPDATA%/PowerX_Keys/TraceData/` | `.dat` binary — `trace_{id}.dat` (MacroRecordingService.cs:139) |
| Debug log | `%LOCALAPPDATA%/PowerXKeys/debug_log.txt` | Text |
| Compiled scripts | `%DOCUMENTS%/PowerX_Keys/Engine/` | .ahk files |

## MVVM Navigation

The app uses a **ViewModel-first navigation** pattern controlled by `MainViewModel`:

- `ScriptLibraryViewModel` — Main dashboard / macro library
- `MacroEditorViewModel` — Macro builder (partial class split across 8 files, incl. `SmartView.cs` and `Optimization.cs`)
- `SettingsDashboardViewModel` — All settings
- `AIAssistantViewModel` — AI chat sidebar

## Critical Files to Know

These are the files you'll touch most often:

- [ScriptCompilerService.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Services/Services/ScriptCompilerService.cs) — The compiler. Generates all AHK code.
- [MacroItem.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Core/Models/MacroItem.cs) — `MacroItem` and `MacroStep` models
- [AppConfig.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Core/Models/AppConfig.cs) — `SettingsModel` and `ActionItem` (hotkey binding)
- [ScriptManager.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Services/Managers/ScriptManager.cs) — AHK process lifecycle
- [MacroDatabase.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Services/Managers/MacroDatabase.cs) — SQLite operations
- [ConfigManager.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Services/Services/ConfigManager.cs) — JSON config load/save

## Common Pitfalls ⚠️

1. **Don't edit the AHK scripts directly** — they're auto-generated. Edit `ScriptCompilerService` instead.
2. **PowerX_Engine.exe is just AHK** — it's a copy of `AutoHotkey64.exe` renamed for Task Manager stealth.
3. **MacroEditorViewModel is a partial class** — split into `Core.cs`, `Properties.cs`, `Commands.cs`, `Recording.cs`, `Capture.cs`, `Optimization.cs`, `SmartView.cs`.
4. **Version is shared now** — update `PowerX_Keys_V2.csproj` and the app UI/runtime version follows through `VersionInfo.cs`.
5. **Config changes need engine reload** — After changing settings or hotkeys, call `ScriptManager.Stop()` then `ScriptManager.Start()` to recompile.
6. **Never force-kill the app** — Kill commands were explicitly removed for compliance (see v5.2.1 changelog).

## How to Add a Feature

See [[adding-a-feature]] for the step-by-step guide, but the quick version:

1. Add properties to the **Model** (`MacroStep`, `ActionItem`, or `SettingsModel`)
2. Add UI in the **View** (XAML)
3. Wire logic in the **ViewModel**
4. If it affects execution: update `ScriptCompilerService` to generate the AHK code
5. Test by running a macro and checking the generated `.ahk` file

## Key Files

- [MacroEditorViewModel.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/ViewModels/MacroEditorViewModel.cs)
- [ScriptLibraryViewModel.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/ViewModels/ScriptLibraryViewModel.cs)
- [MainViewModel.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/ViewModels/MainViewModel.cs)
- [ScriptCompilerService.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Services/Services/ScriptCompilerService.cs)
- [ScriptManager.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Services/Managers/ScriptManager.cs)

## Related Pages

- [[adding-a-feature]]
- [[branching-strategy]]
- [[execution-pipeline]]
- [[macro-editor]]
- [[script-library]]
- [[app-config]]
- [[macro-item]]
- [[database-schema]]
