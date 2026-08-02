---
tags: [manager, ahk, engine, process-lifecycle]
date: 2026-06-12
last_updated: 2026-08-01
sources:
  - PowerX.Services/Managers/ScriptManager.cs
  - PowerX.UI/ViewModels/MainViewModel.cs
status: current
---

# Script Manager ⚙️

`ScriptManager` is the static class that manages the lifecycle of all AutoHotkey processes. It's the bridge between the C# application and the AHK execution engine.

## Overview

- Manages **6 distinct AHK process slots**
- Handles process creation, monitoring, and termination
- White-labels AHK as `PowerX_Engine.exe` for Task Manager stealth
- Kills orphaned processes on startup
- Assigns every AHK process to a **Windows Job Object** so all children die with the app

## Process Slots

| Key | Script | Purpose |
|-----|--------|---------|
| `master` | `master_script.ahk` | The main hotkey engine (permanent foundation pillar) |
| `executor` | `executor_script.ahk` | Separate execution worker process |
| `macro` | — | Single-step preview / test execution |
| `tester` | `test_runner.ahk` | One-off script testing (sandbox) |
| `recorder` | `record_engine.ahk` | Mouse/keyboard recording capture |
| `snippets` | `snippets_script.ahk` | Text snippets engine (conditional) |

The engine is considered "Running" when the `master` process is alive (`IsRunning` → `KEY_MASTER`).

## Process Tracking

Uses a `Dictionary<string, Process>` guarded by a private `lock` object:

```csharp
private static readonly Dictionary<string, Process> _processes = new Dictionary<string, Process>();
```

Each slot can hold one `System.Diagnostics.Process` instance. Starting a new process in a slot automatically kills the previous one.

## Engine Lifecycle

### Start Flow

```
ScriptManager.Start()
  → ScriptCompilerService.CompileMasterScript()   // Generate .ahk from JSON + SQLite
  → ScriptCompilerService.CompileSnippetsScript() // Generate snippets_script.ahk
  → Thread.Sleep(200)                              // Let Windows Defender release locks
  → StartProcess(KEY_MASTER, "master_script.ahk")  // with retry on first failure
  → if master OK: StartProcess(KEY_EXECUTOR, "executor_script.ahk")  // separate worker
  → if master OK AND "TextSnippets" in RunningProfiles AND snippets_script.ahk exists:
      StartProcess(KEY_SNIPPETS, "snippets_script.ahk")
```

- The **master and executor are launched as separate processes** — the executor only starts if the master started successfully.
- The **snippets engine starts only when the TextSnippets profile is explicitly active** (checked via `ServicesUIHooks.GetRunningProfilesHook`), and only if `snippets_script.ahk` exists in `%DOCUMENTS%/PowerX_Keys/Engine/`.

### Stop Flow

```
ScriptManager.Stop()
  → StopProcess(KEY_SNIPPETS)   // Stop snippets first
  → StopProcess(KEY_EXECUTOR)   // Stop executor
  → StopProcess(KEY_MASTER)     // Master last (triggers EngineExited)
```

`StopAllProcesses()` additionally stops `tester`, `recorder`, and `macro`.

### Reload Flow

```
ScriptManager.Reload()
  → Set IsReloading = true
  → Stop()
  → Thread.Sleep(100)
  → Start()
  → Set IsReloading = false
```

`Reload()` is simply **Stop + Start** with an `IsReloading` flag so transient stops during restart don't clear UI state.

## White-Labeling

`GetAhkExecutable()` implements the white-labeling strategy:

1. Look for `CoreData/AutoHotkey64.exe` in app directory
2. Fall back to standard install paths (`C:\Program Files\AutoHotkey\...`)
3. Copy the found executable as `PowerX_Engine.exe` in the app directory
4. Return `PowerX_Engine.exe` path

This means the process appears as "PowerX_Engine" in Task Manager, not "AutoHotkey64".

## Stale Process Cleanup

On static constructor (and `ProcessExit`), `KillStaleProcesses()` sweeps for orphaned processes:
- Targets: `powerx_engine`, `autohotkey64`, `autohotkey`
- Only kills processes launched from the app's directory
- Always kills `powerx_engine` regardless of path
- Prevents zombie engines from accumulating across restarts

## AHK Executable Resolution Order

1. `{AppDir}/CoreData/AutoHotkey64.exe` (bundled)
2. `C:\Program Files\AutoHotkey\v2\AutoHotkey64.exe`
3. `C:\Program Files\AutoHotkey\AutoHotkey.exe`
4. `C:\Program Files\AutoHotkey\AutoHotkey64.exe`
5. `C:\Program Files\AutoHotkey\UX\AutoHotkeyUX.exe`
6. `AutoHotkey64.exe` (PATH fallback)

## Job Object

On startup, `InitJobObject()` creates a Windows Job Object with `JOB_OBJECT_LIMIT_KILL_ON_JOB_CLOSE`. Every AHK process started via `StartProcess()` is attached (`AttachToJobObject`), so if the app dies, **all child AHK processes die automatically** — no orphans left behind.

## Process Priority

⚠️ **`ScriptManager` does NOT set process priority** — no C#-side priority management exists. The priority boost is handled **inside the compiled AHK script** (Turbo Engine Mode, see [[overview]]): each executed macro step calls `ProcessSetPriority("High")` with a rolling 3-second decay timer back to `Normal`. Never uses `Realtime`.

## Events

- `EngineExited` — fired when the master process exits (unexpected crash or intentional stop)
- `KillSwitchActivated` — fired after the engine auto-restarts from a Kill Switch (AHK exit code 10)
- Used by `ScriptLibraryViewModel` to sync the UI toggle state
- **Race Condition Fix (2026-06-12):** Fixed a bug where stopping the engine triggered a concurrent hot-reload process restart. The backing field `_isEngineRunning` in `MainViewModel` is now set to `false` immediately on stopping to prevent hot-reload checks from evaluating to `true`.
- **Reload & UI Sync Fix (2026-06-12):** Fixed race conditions where transient engine stops during restarts cleared `RunningProfiles` and broke status button rendering:
  - Added early checks for `ScriptManager.IsReloading` inside all `EngineExited` subscribers to bypass transient resets.
  - Replaced manual stop-and-start flows with `ScriptManager.Reload()`, ensuring `IsReloading` is flagged.
  - Fixed a typo checking for `"RunningProfile"` (singular) instead of `"RunningProfiles"` (plural) in `MainWindow`'s property listener.
  - Linked `ScriptLibraryViewModel.IsEngineRunning` directly to `MainViewModel.Instance.IsEngineRunning` to keep library views updated.
  - Added `IsEngineOperationInProgress` property to `MainViewModel` to guard against dispatcher race conditions during rapid user-initiated consecutive start/stop clicks.

## Key Files

- [ScriptManager.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Services/Managers/ScriptManager.cs)

## Related Pages

- [[execution-pipeline]]
- [[script-library]]
- [[agent-onboarding]]
- [[macro-editor]]
