---
tags: [service, config, persistence, json, state-management]
date: 2026-05-31
sources:
  - Services/ConfigManager.cs
status: complete
---

# Config Manager

Static singleton responsible for **all application state persistence**. Manages JSON config, debounced auto-save, profile migration, execution stats syncing, and garbage collection of orphaned files.

## Purpose

- Loads/saves `config.json` from `%LOCALAPPDATA%\PowerXKeys\`
- 500ms debounced auto-save to prevent UI hitching during rapid changes
- Safe-write mechanism (temp → replace → backup) to prevent corruption
- Syncs execution stats between C# and the background AHK engine
- Runs garbage collection for orphaned trace data, images, and macro bindings
- Auto-reloads the AHK engine on config changes

## Storage

| Item | Location |
|------|----------|
| Config file | `%LOCALAPPDATA%\PowerXKeys\config.json` |
| Backup | `config.json.bak` (auto-created on each save) |
| Stats bridge | `%TEMP%\PowerX_MacroStats.txt` |
| Trace data | `%APPDATA%\PowerX_Keys\TraceData\` |
| Engine images | `%LOCALAPPDATA%\PowerXKeys\Engine\Images\` |

## Save Strategy

```mermaid
graph LR
    A["Save() called"] -->|500ms debounce| B["ExecuteSave()"]
    B --> C["Serialize on UI thread"]
    C --> D["Write to .tmp"]
    D --> E["File.Replace → .json + .bak"]
    E --> F["ConfigUpdated event"]
    F --> G["Auto-reload AHK engine"]
```

- `Save()` — resets the debounce timer to 500ms
- `ForceSave()` — cancels debounce, writes immediately
- `ExecuteSave()` — thread-safe serialization inside `lock(_saveLock)`
- UI thread serialization prevents "Collection Modified" exceptions

## IPC Stats Syncing

The AHK engine tracks `PendingExecutions` and auto-flushes every 5 minutes to `PowerX_MacroStats.txt`. The C# side:

1. `TriggerOnDemandStatsFlush()` — broadcasts `RegisterWindowMessage("PowerX_FlushStats")` via `HWND_BROADCAST`
2. Waits 150ms for AHK to write the file
3. `FlushMacroStats()` — reads the file, sums all line values, increments `TotalMacrosExecuted`, deletes file

A background `_statsTimer` runs every 5 minutes to auto-sync.

## Profile Auto-Repair

On initialization, the config manager:
- Merges legacy profile names: `"Custom Actions"` → `"CustomActions"`, `"Custom Macros"` → `"MacroBindings"`
- Ensures `Default`, `CustomActions`, `MacroBindings` profiles always exist
- Migrates hotkey profile assignments (one-time flag: `HasMigratedProfileAssignments`)

## Default Hotkey Scaffolding

If hotkeys list is empty, creates defaults for:
- App Bound (pre-configured with **Snipping Tool** `snippingtool.exe` on F10), File Launchers (Notepad on F11), and Custom Keystrokes (Task Manager on F12)
- Media Controls (Volume Up/Down/50%/Mute)
- Media Playback (Play/Pause, Next/Prev Track)
- Browser Tabs & Navigation
- System Navigation (Virtual Desktops)
- Quick Actions (Lock PC, Screenshot, Show Desktop)

## Garbage Collection

`RunGarbageCollection()` runs silently on startup in a background task:

| Target | Cleanup Logic |
|--------|--------------|
| **Trace files** (`.dat`) | Deletes files not referenced by any macro's `TraceFileId` |
| **Images** (`.png/.jpg`) | Deletes files not referenced by any macro's `SearchImageFilename` or hotkey's `TriggerImage` |
| **Ghost macro bindings** | Removes hotkey entries where `Path` (macro ID) doesn't exist in the database |

## Selective Reset

`ResetConfigSelective()` supports granular reset:
- **Settings** — preserves API keys, launch count, tip state
- **All Macros** — clears hotkey bindings, deletes all macros from SQLite
- **AI Data** — clears API key and AI generation count
- **App Stats** — resets launch count, tip state, execution count

## Key Methods

| Method | Description |
|--------|-------------|
| `Initialize()` | Loads config, runs migrations, starts stats timer |
| `Save()` | Debounced save (500ms) |
| `ForceSave()` | Immediate save |
| `FlushMacroStats()` | Reads AHK stats file, updates totals |
| `TriggerOnDemandStatsFlush()` | IPC broadcast to AHK engine |
| `RunGarbageCollection()` | Cleans orphaned traces, images, bindings |
| `ResetConfigSelective()` | Granular factory reset |

## Key Files

- [ConfigManager.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Services/ConfigManager.cs) — 583 lines

## Related Pages

- [[script-compiler]]
- [[macro-execution]]
- [[auto-update]]
- [[ai-assistant]]
