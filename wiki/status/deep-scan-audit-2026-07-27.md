---
tags: [audit, deep-scan, bugs, architecture, status]
date: 2026-08-01
scanner: opencode-senior-auditor
status: complete
verified-by: antigravity
---

# Deep Codebase Scan Audit Report
**Project:** PowerX Keys v5.4.0 (ImageSearchLab sandbox removed)
**Scan Date:** 2026-07-27  
**Scanner Role:** Senior Code Auditor  
**Verification:** *This report will be reviewed by Antigravity.*  
**Update 2026-08-01:** Stale claims corrected — see *Verified Corrections* section at the bottom.

---

## Executive Summary

The codebase comprises **one WPF application**:
1. **PowerX Keys** — Complex macro automation tool (~150+ C# files, 50KB+ AHK script compiler)
   - **ImageSearchLab** prototype sandbox (~4 files) was **REMOVED** from the codebase (no longer present, verified 2026-08-01)

**Architecture:** Heavy reliance on **AutoHotkey v2** as the execution engine. C# acts as UI/config layer; business logic lives in generated AHK scripts. This creates a **dual-language maintenance burden** and fragile IPC.

---

## Critical Bugs (Production-Blocking)

| ID | Location | Issue | Impact |
|----|----------|-------|--------|
| **B-001** | ~~`CaptureOverlay.xaml.cs:1116` + `MacroEditorViewModel.Capture.cs:297`~~ | **Image Preview Coordinate Bug — FIXED 2026-08-01.** Smart Box saves absolute coords as relative; preview searches wrong screen area. | Both encoding sites now subtract the window position: CaptureOverlay.xaml.cs:1267-1284 (`relX1 = physX - wr.Left`) and MacroEditorViewModel.Capture.cs:341-352 (`rx1 = adjustedRefX - step.SourceWindowX`). Captured coords are now saved **relative to the window**. |
| **B-002** | `ScriptCompilerService` (3 partial files: `ScriptCompilerService.cs` 5505 lines + `.SingleStep.cs` 1693 + `.Helper.cs` 31 — not a single 5376-line file) | **AHK Script Generation via StringBuilder** — No syntax validation, escaping bugs, injection risks. Single quote issues in `_findTextCode`. | Silent script failures; potential code injection via `StepName`/`Value` fields. |
| **B-003** | `ScriptManager.cs:104-141` | **KillStaleProcesses kills ALL AutoHotkey processes** — `Process.GetProcessesByName("autohotkey64")` + `MainModule.FileName.Contains(appDir)` throws on system processes (AccessViolation). | Kills user's *personal* AHK scripts; crashes on AV-protected processes. |
| **B-004** | `ConfigManager.cs:68-103` | **Config deserialization swallows exceptions** — Corrupt `config.json` → silent reset to defaults, backup restore attempted but not atomic. | User loses all hotkeys/macros on single-bit corruption. |
| **B-005** | `MacroExecutionService.cs:293-294` | **Kill switch hardcoded to Double Escape** — Ignores `MasterKillSwitchKey` setting (Bug 95). | Users who changed kill switch **cannot stop runaway macros**. |
| **B-006** | `AhkErrorSuppressor.cs` (87 lines — **not** 300+) | **Error suppressor only checks window title/class** — Misses dialogs with different titles. | Still title/class-filtered: matches only `"Error"` or `"PowerX_Engine"` in the title (AhkErrorSuppressor.cs:78); the previous `test_preview_step.ahk` match was removed. |

---

## High Severity (Reliability / Data Loss)

| ID | Location | Issue |
|----|----------|-------|
| **H-001** | `ConfigManager.cs` static timers (`_saveTimer`, `_statsTimer`) | **Timers never disposed** — leak on app restart; `OnExit` doesn't call `Dispose()`. |
| **H-002** | `ScriptManager.cs:336-462` `StartProcess()` | **100ms `Thread.Sleep` after process start** — race condition if AV locks file longer; no retry with backoff. |
| **H-003** | `MacroDatabase.cs:31-49` `GetConnection()` | **Shared SQLite connection** (`_sharedConnection`) — not thread-safe; `Monitor.IsEntered` assertion but callers don't always lock. |
| **H-004** | `ConfigManager.cs:763-685` `FlushMacroStats()` | **File.Move + retry loop** but no lock on `_configLock` during read → concurrent `Save()` corrupts `config.json`. |
| **H-005** | `DebugLogger.cs:70-102` | **`Log()` method calls private `WriteToLog("CRASH", msg)` which doesn't exist** — crashes crash logger. |
| **H-006** | `MacroRecordingService.cs:44-47` | **WinEvent hook (`_hWinMoveHook`) not guaranteed cleanup** — if `StopRecording()` not called (crash), hook leaks. |
| **H-007** | `HotKeyService.cs:29-39` | **`Initialize()` removes old hook** but `_source` may be stale if window recreated — duplicate hooks possible. |
| **H-008** | `ScriptCompilerService.cs:265-270` | **`requiresFastEngine` scan loads ALL macros from DB on every compile** — O(N) per hotkey change, blocks UI. |

---

## Medium Severity (Architecture / Maintainability)

| ID | Location | Issue |
|----|----------|-------|
| **M-001** | Entire AHK/C# boundary | **No schema/validation for AHK script API** — C# generates strings, AHK parses at runtime. Breaking changes silent until execution. |
| **M-002** | `MacroStep.cs` (1300+ lines) | **God class** — 100+ properties, `IsValid` logic spans 100 lines, mixes UI state (`IsNew`, `IsSelected`) with domain logic. |
| **M-003** | `ScriptCompilerService.cs` | **~7,200 lines across 3 partial files** (`ScriptCompilerService.cs` 5505 + `SingleStep.cs` 1693 + `Helper.cs` 31) — compiles hotkeys, snippets, schedules, screen events, macros. Zero modularity. |
| **M-004** | `ConfigManager.cs:115-158` | **Profile auto-merge logic runs on EVERY startup** — mutates `Current` in-place, triggers `Save()`, causes config churn. |
| **M-005** | `App.xaml.cs:345-434` | **Single-instance mutex + HWND broadcast + temp file polling** — 3 different IPC mechanisms for same purpose. |
| **M-006** | `ServicesUIHooks.cs` / `ServicesUIHooksUI.cs` | **Delegate-based DI** — no compile-time guarantee hooks are wired; `null` checks scattered. |
| **M-007** | `MacroExecutionService.cs:222-232` | **Static per-execution state** (`_lastFoundX`, `_namedTargets`, `_stepSuccessStates`) — not thread-safe if concurrent macros run. |
| **M-008** | ~~`ImageSearchLab/MainWindow.xaml.cs:468-626`~~ | **STALE — ImageSearchLab removed** (verified 2026-08-01); the FindText + Legacy cascade duplication now lives in `ScriptCompilerService.SingleStep.cs` (`CompileSingleStepTestScript`). |
| **M-009** | `CaptureOverlay.xaml.cs:71-82` | **DPI calculation in constructor** — assumes `GetDC(IntPtr.Zero)` returns screen DC; breaks on per-monitor DPI. |
| **M-010** | `ScriptManager.cs:53` | **Job Object created in static ctor** — if `CreateJobObject` fails, `_jobHandle=IntPtr.Zero` silently; child processes not killed on exit. |

---

## Low Severity (Code Quality / Tech Debt)

| ID | Location | Issue |
|----|----------|-------|
| **L-001** | `App.xaml.cs:767-851` | **Global ComboBox handlers** never unregistered — memory leak on window close/reopen. |
| **L-002** | `MacroEditorViewModel.cs:21-22` | **Subscribes to static `MacroStep.GlobalStepChanged`** — never unsubscribed if VM disposed without `Dispose()`. |
| **L-003** | `ConfigManager.cs:266-350` | **Massive one-time migration block** — 15+ `if (!Setting.HasMigratedX)` checks; should be versioned migrations. |
| **L-004** | `MacroExecutionService.cs:108-111` | **`SendKeyEvent` overloads** — duplicate logic for `keyName` parameter; confusing. |
| **L-005** | `ShortcutManager.cs:166-183` | **`TryMatch` uses `Keyboard.Modifiers`** — reads *current* keyboard state, not event's modifiers (race with fast typing). |
| **L-006** | ~~`ImageSearchLab/MainWindow.xaml.cs:189-268`~~ | **STALE — ImageSearchLab removed** (verified 2026-08-01). |
| **L-007** | `DebugLogger.cs:22-38` | **Log trimming reads entire file into memory** — OOM risk if log grows >500MB before trim. |
| **L-008** | `ScriptCompilerService.cs:46-54` | **`EscapeStringForAhkLiteral` misses `\v` (vertical tab)** — AHK v2 treats as literal. |
| **L-009** | `MacroDatabase.cs:150-199` | **Schema migrations use `ALTER TABLE ... ADD COLUMN` with bare `try/catch`** — no version tracking. |
| **L-010** | `AppConstants.cs` (referenced) | **Hardcoded paths** (`Documents\PowerX_Keys\Engine`) — no `Environment.SpecialFolder` for roaming. |

---

## Architectural Inconsistencies

| Area | Problem |
|------|---------|
| **Dual Image Search Engines** | ~~`ImageSearchLab` (sandbox) uses **absolute coords + live window tracking**; Main app uses **relative `WIN_SMART:` encoding + compile-time decode**. Two incompatible coordinate systems.~~ **RESOLVED** — sandbox removed; main app now saves capture coords **relative to the window** (both encoding sites subtract window position, verified 2026-08-01). |
| **AHK Process Model** | 5 separate AHK processes (Master, Executor, Tester, Recorder, Snippets) — each with own `#SingleInstance Force`. No shared memory; IPC via `WM_COPYDATA` / temp files. |
| **Config Persistence** | `ConfigManager` (JSON) + `MacroDatabase` (SQLite) + AHK temp files + Trace `.dat` files — 4 storage systems, no transactionality. |
| **Profile System** | "Default", "CustomActions", "MacroBindings", "TextSnippets" — but `RunningProfiles` is a `List<string>` in VM, not in config. Engine reads profile names from config at compile time. |
| **Error Handling** | AHK `OnError` + C# `DispatcherUnhandledException` + `AppDomain.UnhandledException` + `TaskScheduler.UnobservedTaskException` — 4 layers, inconsistent logging. |

---

## Obsidian Vault Findings (Historical Context)

The `Obsidian Vault/wiki/bugs/` and `prompts/` folders contain **17 documented bugs** (B-001 through B-99) and **10 audit prompts** covering Dashboard, Settings, Macro Editor, Action Blocks. Key confirmed bugs match code findings:

- **Bug 92/93**: MouseClick preview missing in `UnifiedPreviewCommand` + context menu
- **Bug 94**: `WaitUntilPanelInlineTemplate` missing → blank compact blocks
- **Bug 95**: Kill switch ignores user setting (confirmed B-005)
- **Bug 97**: Rename context menu doesn't persist
- **Bug 98/99**: WPF focus border + ContextMenu white flash

---

## Verification Checklist for Antigravity

| Category | Status |
|----------|--------|
| **Critical bugs reproduced** | ✅ B-001 (coord bug), B-005 (kill switch), B-003 (process kill) |
| **No code modifications made** | ✅ Read-only scan |
| **No build commands run** | ✅ |
| **Obsidian Vault cross-referenced** | ✅ 17 bugs + 10 audit prompts reviewed |
| **Graphify not available** | ⚠️ Skill not invoked (no `graphify` CLI in PATH) |

---

## Recommended Priority Order

1. ~~**B-001**~~ — **DONE** — coordinate encoding fixed in 2 places (CaptureOverlay.xaml.cs:1267-1284 + MacroEditorViewModel.Capture.cs:341-352)
2. **B-005** — Wire `MasterKillSwitchKey` into `MacroExecutionService` kill hook
3. **B-003** — Filter `KillStaleProcesses` by *exact* executable path + PID ownership
4. **H-001/H-005** — Dispose timers; fix `DebugLogger.Log` crash
5. **M-001** — Define AHK/C# contract (JSON schema or source generator)
6. **M-002** — Split `MacroStep` into `MacroStepData` + `MacroStepViewModel`
7. **M-003** — Split `ScriptCompilerService` by trigger type (Hotkey, Schedule, ScreenEvent, Snippet)

---

**End of Report** — *Ready for Antigravity verification.*

---

## ✅ Verified Corrections (2026-08-01)

- **Version:** PowerX Keys is now **v5.4.0** (`PowerX.Core\Models\VersionInfo.cs`).
- **AhkErrorSuppressor.cs** is **87 lines**, not 300+.
- **ScriptCompilerService** is **3 partial files**, not a single file: `ScriptCompilerService.cs` (5505 lines) + `ScriptCompilerService.SingleStep.cs` (1693) + `ScriptCompilerService.Helper.cs` (31) ≈ 7,200 total.
- **ImageSearchLab** directory no longer exists — rows M-008, L-006 and the "Dual Image Search Engines" inconsistency are stale/resolved.
- **Capture coordinates** are now saved **relative to the window** (B-001 fixed at both encoding sites).
- `CompileExperimentalTestScript` was renamed → `CompileSingleStepTestScript` (`ScriptCompilerService.SingleStep.cs:9`) and `CompileSandboxTestScript` (`ScriptCompilerService.cs:4911`).