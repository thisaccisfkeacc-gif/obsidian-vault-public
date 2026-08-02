---
tags:
  - architecture
  - execution
  - compilation
  - ahk
date: 2026-05-23
sources:
  - Services/ScriptCompilerService.cs
  - Managers/ScriptManager.cs
status: complete
---

# Execution Pipeline

This page documents how macros flow from user configuration all the way to live execution via AutoHotkey.

## Pipeline Overview

```
JSON Config (AppConfig)
    ↓
ScriptCompilerService.CompileMasterScript()
    ↓
[master_script.ahk (Listener)] ──(Win32 PostMessage 0x5555)──> [executor_script.ahk (Executor)]
    ↓                                                             ↓
ScriptManager.Start()                                        ScriptManager.Start()
    ↓                                                             ↓
PowerX_Engine.exe (Listener Process)                         PowerX_Engine.exe (Executor Process)
    ↓                                                             ↓
Live hotkey interception                                     Native macro steps execution
```

## Stage 1: JSON Configuration

All user-configured actions live in `ConfigManager.Current` as an `AppConfig` object containing:
- `Hotkeys` — List of `ActionItem` objects (key bindings, trigger modes, function, path to macro)
- `Settings` — Global settings (kill switch key, performance mode, etc.)
- `Profiles` — Multi-profile support (CustomActions, MacroBindings, user-created profiles)

Macros themselves are stored separately in **SQLite** via `MacroDatabase` — the `ActionItem.Path` field holds the macro's GUID string.

## Stage 2: Script Compilation

[ScriptCompilerService.CompileMasterScript()](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Services/ScriptCompilerService.cs#L11) is the heart of the pipeline. It's a **2,227-line** static partial class that generates a complete AHK v2 script from the current config state.

### Compilation Steps

1. **Read config** — Grab `ConfigManager.Current` hotkeys and profiles
2. **Load all macros** — `MacroDatabase.LoadAllMacros()` to resolve macro GUIDs
3. **Dependency scan** — Check if any macro step uses `ImageSearch` with `UseFastEngine` → if so, include `FindText.ahk`
4. **Generate AHK header**:
   - `#Requires AutoHotkey v2.0`
   - `SetTitleMatchMode 2` (substring matching)
   - `CoordMode "Pixel", "Screen"` + `CoordMode "Mouse", "Screen"`
   - `#SingleInstance Force`, `#NoTrayIcon`, `Persistent()`
5. **Generate kill switch** — Configurable panic button (default: Double Escape)
6. **Generate ghost keyboard reset** — `OnExit` handler that releases all stuck modifier keys
7. **Generate performance tracker** — `PendingExecutions` counter with 5-minute auto-flush via `SetTimer`
8. **Filter active hotkeys** — Only enabled, non-conflicting, profile-matched actions with valid keys
9. **Compile each action** — Based on `TriggerMode`:
   - **Hotkey** (default) — Standard AHK hotkey binding
   - **Schedule** — `SetTimer()` with interval, optional run-on-start
   - **ScreenEvent** — Polling `ImageSearch`/`PixelSearch` with configurable bounds
   - **Toggle** — Same key cycles through 2-5 macros with timeout reset
   - **MobileRemote** — Skipped (handled by HTTP server, not AHK)
10. **Write to disk** — `master_script.ahk` (Listener) and `executor_script.ahk` (Executor) in the Engine folder

### Output Location

```
%USERPROFILE%\Documents\PowerX_Keys\Engine\master_script.ahk (Listener)
%USERPROFILE%\Documents\PowerX_Keys\Engine\executor_script.ahk (Executor)
```

Other engine-related files in the same folder:
- `FindText.ahk` — Fast pattern recognition library (copied if needed)

### Scoping Rules

Each action can be scoped to specific applications:
- **Include mode** → `GroupAdd` + `#HotIf WinActive("ahk_group IncludeGroupXXXX")`
- **Exclude mode** → `GroupAdd` + `#HotIf !WinActive("ahk_group ExcludeGroupXXXX")`
- **Global** → `#HotIf ; Global Scope`

### Macro Step Types Compiled to AHK

| Step Type | AHK Output |
|---|---|
| `Delay` | `Sleep(N)` |
| `Text` | `SendEvent("{Text}" . "...")` or clipboard-paste (`^v`) for FastPaste |
| `Keyboard` | `Send("{Blind}" . "{key}")` with Hold Down/Release Up variants |
| `MouseClick` | `Click(x, y, "Button")`, scroll, move-only |
| `Popup` | `MsgBox("...", "...")` |
| `Notification` | `TrayTip "...", "...", 1` |
| `UserInput` | `InputBox()` / custom GUI with dropdown/yes-no |
| `WaitForKey` | Polling loop with `GetKeyState()` |
| `FileLauncher` | `Run("path")` |
| `SystemSound` | `SoundPlay("*-1")` |
| `LoopSequence` | `Loop N { ... }` with recursive child compilation |
| `ImageSearch` | `ImageSearch(&FoundX, &FoundY, ...)` |
| `PixelSearch` | `PixelSearch(&FoundX, &FoundY, ...)` |

## Stage 3: Engine Launch

[ScriptManager.Start()](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Managers/ScriptManager.cs#L94) handles the process lifecycle:

1. **Compile** — Calls `ScriptCompilerService.CompileMasterScript()`
2. **AV delay** — Sleeps 800ms to let Windows Defender release file locks
3. **Resolve executable** — `GetAhkExecutable()` finds or creates `PowerX_Engine.exe`
4. **Launch process** — `Process.Start()` with `CreateNoWindow=true`, `UseShellExecute=false`
5. **Verify alive** — Wait 100ms then check `HasExited` to catch immediate crashes
6. **Retry** — If first attempt fails, wait 500ms and try once more

> 💡 **Turbo Engine Mode** (default OFF) is handled inside the compiled script, not here: each compiled step emits `TurboBoost()` which sets `ProcessPriorityClass.High` and resets a 3s decay timer; the engine returns to `Normal` 3s after the last step. Toggling it saves config and reloads the engine.

### 4 Micro-Services Architecture

`ScriptManager` manages **5 distinct AHK process slots**:

| Key | Script | Purpose |
|---|---|---|
| `master` | `master_script.ahk` | Primary hotkey listener process |
| `executor` | `executor_script.ahk` | Dedicated macro execution process |
| `macro` | — | Reserved slot |
| `tester` | `test_runner.ahk` | Sandbox testing (preview mode) |
| `recorder` | `record_engine.ahk` | Macro recording capture |

Only `master` and `executor` use the Documents-based Engine folder. Others use internal `Scripts/` folder.

### White-Label Process

`GetAhkExecutable()` in [ScriptManager.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Managers/ScriptManager.cs#L74):

1. Look for `CoreData/AutoHotkey64.exe` in app directory
2. Fallback to `C:\Program Files\AutoHotkey\v2\AutoHotkey64.exe` and other paths
3. Copy the found binary to `PowerX_Engine.exe` (hides "AutoHotkey" in Task Manager)
4. Return `PowerX_Engine.exe` path

### Engine Stop & Stats Flush

When stopping the engine ([ScriptManager.Stop()](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Managers/ScriptManager.cs#L120)):

1. Send `PowerX_FlushStats` Windows message to AHK
2. Wait 200ms for AHK to write pending stats to `%TEMP%\PowerX_MacroStats.txt`
3. Kill the process
4. Fire `EngineExited` event to update UI

### Stale Process Cleanup

On static construction, `ScriptManager` kills any orphaned processes:
- Targets: `powerx_engine`, `autohotkey64`, `autohotkey`
- Only kills processes launched from the app's directory (avoids killing user's personal AHK scripts! 🛡️)

## Stage 4: Live Execution

Once `master_script.ahk` is running via `PowerX_Engine.exe`:
- AHK intercepts configured hotkeys at the OS level
- Macro steps execute natively in AHK (mouse, keyboard, delays, logic)
- Stats counter increments `PendingExecutions` per execution
- Stats flushed to temp file every 5 minutes or on-demand via IPC
- Kill switch (default: Double Escape) calls `ExitApp`

## Key Files

- [ScriptCompilerService.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Services/ScriptCompilerService.cs) — The 2,227-line compiler
- [ScriptCompilerService.SingleStep.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Services/ScriptCompilerService.SingleStep.cs) — Single-step test script compiler
- [ScriptManager.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Managers/ScriptManager.cs) — Process lifecycle manager
- [ConfigManager.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Services/ConfigManager.cs) — JSON config source

## Related Pages

- [[overview]] — Full architecture overview
- [[dual-execution-model]] — AHK vs C# execution comparison
- [[component-relationships]] — How compiler fits into the system
