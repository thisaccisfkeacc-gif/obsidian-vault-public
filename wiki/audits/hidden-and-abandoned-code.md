# Hidden, Abandoned & Leftover Code Audit

**Audit Date:** July 25, 2026
**Scope:** PowerX.Core, PowerX.Services, PowerX.UI, PowerX_Keys_V2, WebUI, Themes, Resources, Scripts
**Type:** Read-only scan — no code changes

---

## 📌 Easy Overview (Plain English)

Here is the updated status of all audited items:

- ✅ 🔘 **Secret Test Button on Main Screen** — **REMOVED** (Cleaned up from `MainWindow.xaml`).
- ✅ 🧪 **Debug Checkpoint Cleaner** — **REMOVED** (Cleaned up from `MacroExecutionService.cs` & `MainViewModel.cs`).
- ✅ 🔑 **Free AI Test Switch (DEV_MODE)** — **REMOVED** (Cleaned up from `AIFallbackService.cs`).
- ✅ 💳 **Mock Subscription Test File** — **REMOVED** (Cleaned up from `SupabaseAuthService.cs`).
- ✅ 🖼️ **Fake / Unfinished Web Editor Screen** — **REMOVED** (Deleted `WebUI/MacroEditorV2/` prototype folder).
- ✅ 📜 **Dead Helper Scripts** — **REMOVED** (Deleted obsolete `test_runner.ahk`).
- ✅ 🔒 **Hidden "Trial Remaining" Card** — **REMOVED** (Cleaned up collapsed grid from `MainWindow.xaml`).
- ✅ 🎨 **Fixed Colors Tokens** — **COMPLETED** (Created all 23 missing color tokens in `DarkTheme.xaml` & `LightTheme.xaml`).
- 📄 **Debug Log Text Files** — **KEPT** (Preserved debug log generation as requested).

---

## Summary

| Project | Findings | Critical | High | Medium | Low |
|---------|----------|----------|------|--------|-----|
| PowerX.Core | 30 | 2 | 8 | 12 | 8 |
| PowerX.Services | 25 | 3 | 8 | 10 | 4 |
| PowerX.UI | 100+ | 3 | 8 | 15 | 74+ |
| PowerX_Keys_V2 | 48 | 2 | 6 | 8 | 32 |
| WebUI | 11 | 3 | 5 | 3 | 0 |
| Themes | 7 | 0 | 3 | 4 | 0 |
| Resources | 4 | 0 | 0 | 4 | 0 |
| Scripts | 8 | 0 | 4 | 4 | 0 |
| **TOTAL** | **230+** | **13** | **42** | **60** | **118+** |

---

## 🔴 CRITICAL — Remove Before Packaging

| # | Project | File | Line(s) | Description |
|---|---------|------|---------|-------------|
| 1 | PowerX_Keys_V2 | `MainWindow.xaml` | 953-960 (old) | ~~"Jump to Next Checkpoint" test button~~ — **REMOVED** (no matches in code, verified 2026-08-01) |
| 2 | PowerX_Keys_V2 | `MainWindow.xaml` | 962-976 (old) | ~~Hidden "Trial Remaining" stats card~~ — **REMOVED** (no matches in code, verified 2026-08-01) |
| 3 | PowerX.Services | `MacroExecutionService.cs` | 3302-3342 (old) | ~~`DismissCurrentCheckpoint()`~~ — **REMOVED** (method gone from file, verified 2026-08-01) |
| 4 | PowerX.Services | `AIFallbackService.cs` | 41-58 | `DEV_MODE` bypass — returns fake test macro JSON |
| 5 | PowerX.Services | `SupabaseAuthService.cs` | 267-311 | `sub_mock.json` developer simulation — bypasses real subscription checks |
| 6 | PowerX.Services | `ScriptCompilerService.cs` | 4488-4530 | Writes `master_script_DEBUG.txt` on every compile |
| 7 | PowerX.Services | `ScriptCompilerService.cs` | 4726-4744 | Writes `snippets_script_DEBUG.txt` on every compile |
| 8 | PowerX.UI | `MainViewModel.cs` | 527-528, 580-584 (old) | ~~`JumpToNextCheckpointCommand`~~ — **REMOVED** (no matches in code, verified 2026-08-01) |
| 9 | PowerX.Core | `MacroItem.cs` | 1491-1504 | `SearchImageSourceAppName/WindowTitle` — marked "TEMP: for testing" |
| 10 | PowerX.Core | `MacroItem.cs` | 1700 | `IsDebugHighlight` — has production consumers now (MacroExecutionService.cs:1745-1748, 2670; ScriptCompilerService.cs:2402, 2620, 3612) — **no longer abandoned** |

---

## 🟠 HIGH — Should Fix Soon

### PowerX.Core

| # | File | Line(s) | Type | Description |
|---|------|---------|------|-------------|
| 1 | `AppConfig.cs` | 462-468 | Dead Code | `ShowCapturePrompt` always returns `false` — fully wired but dead |
| 2 | `MacroItem.cs` | 244-245 | Dead Code | `IsKeyCombo => false` — permanent stub |
| 3 | `MacroItem.cs` | 39 | Debug Override | `LogicIfExperimental = 50` — testing-only enum in production |
| 4 | `AppConfig.cs` | 481-482 | Obsolete | `InstanceHWnd` marked `[Obsolete]` — should be removed from serialization |
| 5 | `AppConfig.cs` | 1065 | Commented Code | Task Scheduler section header — dead placeholder |
| 6 | `AppConstants.cs` | 24 | Debug | `DebugLogPath` — debug infrastructure in Core library |
| 7 | `MacroItem.cs` | 60-64 | Unused | `MouseActions` static array — never referenced in Core |
| 8 | `MacroItem.cs` | 149 | Unused | `VirtualSourceSteps` — declared, never read/written |

### PowerX.Services

| # | File | Line(s) | Type | Description |
|---|------|---------|------|-------------|
| 9 | `AIFallbackService.cs` | 21 | Debug Override | `PROXY_URL = ""` — empty, feature non-functional |
| 10 | `TelemetryService.cs` | 23-24 | Debug Override | `SupabaseUrl/AnonKey = ""` — empty, feature non-functional |
| 11 | `AutoUpdateService.cs` | 23 | Debug Override | `UpdateJsonUrl = ""` — empty, feature non-functional |
| 12 | `ScriptCompilerService.SingleStep.cs` | 362 | Debug | `Debug.WriteLine("[PREVIEW DEBUG]")` — left in production |
| 13 | `ScriptCompilerService.SingleStep.cs` | 365-366 | TEMP | Smart Box bounds debug log — temporary diagnostic |
| 14 | `MacroDatabase.cs` | 558, 822 | TEMP | "TEMP: source window info" markers |
| 15 | `SupabaseAuthService.cs` | 442-444 | Stub | Empty interface implementations (IGotrueSessionPersistence) |
| 16 | `SmoothTraceEngine.cs` | 15-18 | Debug | Parallel `DebugLog()` method — not via DebugLogger |

### PowerX.UI

| # | File | Line(s) | Type | Description |
|---|------|---------|------|-------------|
| 17 | `MacroEditorViewModel.Commands.cs.bak` | ALL | Dead File | 1242-line backup file — remove |
| 18 | `SettingsDashboardView.xaml.cs.clean` | ALL | Dead File | Corrupt backup — remove |
| 19 | `SettingsDashboardView.xaml.cs.txt` | ALL | Dead File | Garbled backup — remove |
| 20 | `MacroEditorView.xaml.test` | ALL | Dead File | 1210-line test copy — remove |
| 21 | `SearchTemplates.xaml.temp` | ALL | Dead File | 296-line temp file — remove |
| 22 | `MacroEditorViewModel.Optimization.cs` | 13-14, 49, 187 | Debug | Writes `recorder_debug.log` to disk |
| 23 | `CaptureOverlay.xaml.cs` | 458-463 | Debug | Writes `ImageSearch_Diagnostic.log` to disk |
| 24 | `KeyboardInputTemplates.xaml` | 1260 | Placeholder | `<Setter Text="TEMPORARY"/>` — visible in UI |

### PowerX_Keys_V2

| # | File | Line(s) | Type | Description |
|---|------|---------|------|-------------|
| 25 | `App.xaml.cs` | 240-247 | Debug Override | Clears diagnostic log on every startup |
| 26 | `App.xaml.cs` | 287-289 | Debug Override | Hardcoded Debug build path for updater |
| 27 | `Scripts/MasterScript/config.json` | 66-76 | Debug Override | Hardcoded test hotkeys (Capslock, Win+F1→Notepad) |
| 28 | `build_errors.txt` | ALL | Dead File | Build error log — should not be in source |
| 29 | `task_dashboard.html` | ALL | Dead File | Developer task dashboard — not part of app |
| 30 | `ideas.md` | ALL | Dead File | Stale redirect placeholder |

---

## 🟡 MEDIUM — Clean Up

### PowerX.Core — Stale Migration Flags

These one-shot booleans flip from false→true after migration. After all users migrate, they're dead weight:

| File | Line | Property |
|------|------|----------|
| `AppConfig.cs` | 112 | `HasMigratedAIAssistantHiddenByDefault` |
| `AppConfig.cs` | 158 | `HasMigratedProfileAssignments` |
| `AppConfig.cs` | 159 | `HasAddedStarterMacros` |
| `AppConfig.cs` | 160 | `HasEnabledAllByDefault` |
| `AppConfig.cs` | 161 | `HasMigratedTrayRecordingDefault` |
| `AppConfig.cs` | 202 | `SnippetsSeeded` |
| `AppConfig.cs` | 203 | `MacrosSeeded` |
| `AppConfig.cs` | 204 | `HasMigratedSnippetsV2` |
| `AppConfig.cs` | 205 | `HasMigratedSnippetsV3` |
| `AppConfig.cs` | 206 | `HasMigratedSnippetsV4` |

### PowerX.Core — Unused Types/Members

| File | Line(s) | Item |
|------|---------|------|
| `ActionModels.cs` | 18-23 | `FindTextMode` enum — never referenced in Core |
| `ActionModels.cs` | 25-78 | `DefaultActionModel` class — never used in Core |
| `AppConfig.cs` | 70 | `AdminModeVIP` — developer override flag |
| `AppConfig.cs` | 130 | `RemoteAccessEnabled` — runtime-only toggle |
| `AppConfig.cs` | 1692 | `AppConfig.SearchEngine` — hardcoded, never read |
| `VersionInfo.cs` | 8-10 | `Major/Minor/Patch` — only `FullVersion` is used |
| `MacroItem.cs` | 1633-1638 | `SearchEngine` on MacroStep — never read |
| `MacroItem.cs` | 2424 | `NotifyGlobalStepChanged()` — declared at 2424, called from Name/Icon setters (2476, 2482) — **no longer abandoned** |

### PowerX.Services — Debug Code

| File | Line(s) | Description |
|------|---------|-------------|
| `MacroRecordingService.cs` | 223-230 | `#if DEBUG` block writes `recording_debug.log` |
| `ScriptCompilerService.cs` | 4488-4530 | Debug file write on every compile |
| `ScriptCompilerService.cs` | 4726-4744 | Debug file write on every compile |
| `ScriptCompilerService.SingleStep.cs` | 362 | Debug.WriteLine in production |
| `ScriptCompilerService.SingleStep.cs` | 365-366 | TEMP diagnostic log |
| `MacroExecutionService.cs` | 3302-3342 (old) | ~~TEMPORARY method marked for removal~~ — **REMOVED** (verified 2026-08-01) |

### PowerX.UI — Hidden Features

| File | Line(s) | Description |
|------|---------|-------------|
| `MacroEditorView.xaml` | 1185-1213 | "Playback Speed" panel — hidden, waiting for implementation |
| `MacroEditorView.xaml` | 1655 | "Group Selected" menu item — hidden |
| `ScriptLibraryView.xaml` | 1912-2033 | 6 "COMING SOON" feature cards — disabled |
| `MiscTemplates.xaml` | 470 | `<Setter Text="TEMPORARY PAUSE"/>` — visible text |
| `KeyboardInputTemplates.xaml` | 1260 | `<Setter Text="TEMPORARY"/>` — visible text |

### PowerX_Keys_V2 — Scratch Folder

20 files in `scratch/` — all abandoned test/utility scripts. Already excluded from compilation.

---

## 🟢 LOW — Technical Debt

### PowerX.Core

- `CaptureLibraryEntry.cs:389` — `TrimmedTitle` backward-compat alias
- `MacroItem.cs:432` — Legacy "Background Click" compat remap
- `MacroItem.cs` — Blank line formatting issues (lines 145-147, 1979-1981, 622-624)
- `AppConfig.cs` — 10 stale migration flags (see Medium section)
- `MacroStep.GlobalStepChanged` vs `MacroItem.GlobalStepChanged` — duplicate event names

### PowerX.Services

- `ScriptManager.cs:146,148` — Unused constants `KEY_MACRO`, `KEY_TESTER`
- `SupabaseAuthService.cs:442-444` — Empty interface stubs
- `ScriptCompilerService.cs:1987` — Inline no-op comment

### PowerX.UI

- 25 `NotImplementedException` in converter `ConvertBack` methods — standard WPF pattern
- 100+ empty `catch { }` blocks — silent exception swallowing
- ~80 Collapsed/Hidden elements — normal WPF state toggling
- ~67 conditional visibility bindings — normal WPF pattern
- `MiscTemplates.xaml:470` — "TEMPORARY PAUSE" text visible in UI

### PowerX_Keys_V2

- `Scripts/master_script.ahk` — 11-line stub (actual logic in `MasterScript/` subfolder)
- `Scripts/test_runner.ahk` — Test-only script
- `ideas.md` — Stale redirect
- `MainWindow.xaml.cs:346-349` — Incomplete XML doc comment

---

## WebUI (HTML/WebView2)

### 🔴 CRITICAL

| # | File | Line(s) | Type | Description |
|---|------|---------|------|-------------|
| 1 | `MacroEditorV2/index.html` | ALL | Incomplete | **Zero JavaScript** — pure static HTML mockup, no functionality at all |
| 2 | `Settings/index.html` | 325-328 | Stub | Export/Import/Backup/Restore buttons — no onclick handlers |
| 3 | `Settings/index.html` | 341-345 | Dead Code | `notifySettingChanged()` — defined but never called |

### 🟠 HIGH

| # | File | Line(s) | Type | Description |
|---|------|---------|------|-------------|
| 4 | `Settings/index.html` | 336 | TODO | "Future: receive settings state from C#" — unimplemented |
| 5 | `Settings/index.html` | 337 | Debug | `console.log('Received from C#:')` — left in production |
| 6 | `MacroEditorV2/index.html` | 372-425 | Hardcoded Data | 6 demo timeline cards with fake coordinates/actions |
| 7 | `MacroEditorV2/index.html` | 429-456 | Hardcoded Data | Properties panel entirely static with fake values |
| 8 | `MacroEditorV2/index.html` | 316-325 | Bug | Cards 7+ permanently invisible (opacity:0, no animation-delay) |

### 🟡 MEDIUM

| # | File | Line(s) | Type | Description |
|---|------|---------|------|-------------|
| 9 | `MacroEditor/` | - | Empty Dir | Empty directory — leftover from V1 |
| 10 | `DelayBlock/` | - | Empty Dir | Empty directory — planned but never built |
| 11 | `Settings/index.html` | 181-185 | Unused CSS | `.separator` class defined but never used |

---

## Themes

### 🟠 HIGH

| # | File | Line(s) | Type | Description |
|---|------|---------|------|-------------|
| 12 | `App.xaml` | 279-1362 | Hardcoded Colors | **~97 hardcoded hex colors** across 15+ styles — will break Light Theme |
| 13 | `DesignTokens.xaml` | 22-57 | Unused | **20 structural tokens** (radius, font, spacing, ease) — all never referenced |
| 14 | `App.xaml` | 687 | Duplicate | `GhostButton` style defined twice — App.xaml:687 shadows SettingsStyles.xaml:164 version (line ref updated 2026-08-01) |

### 🟡 MEDIUM

| # | File | Line(s) | Type | Description |
|---|------|---------|------|-------------|
| 15 | `DarkTheme.xaml` + `LightTheme.xaml` | various | Legacy Aliases | 13 unused legacy color/brush aliases (CardBackground, MutedBackground, TextWhite, etc.) |
| 16 | `SettingsStyles.xaml` | 65-158 | Unused | 11 styles defined but never referenced (SettingsPageTitleStyle, DangerZoneCardStyle, etc.) |
| 17 | `App.xaml` | 319-320, 709 | Hardcoded | 4 `StaticResource` instead of `DynamicResource` — won't update on theme switch |
| 18 | `DarkTheme.xaml` + `LightTheme.xaml` | various | Legacy Aliases | 4 legacy aliases still in use (AppBackground, PanelBackground, BorderSubtle, GreenRun*) — should migrate to Token* keys |

---

## Resources

### 🟡 MEDIUM

| # | File | Type | Description |
|---|------|------|-------------|
| 19 | `PowerX_Updater.pdb` | Orphaned | Debug symbols file — not referenced, should not ship |
| 20 | `Sounds/notification.wav` | Large | 374.7 KB — largest WAV, could compress |
| 21 | `Sounds/alert.wav` | Large | 187.4 KB — could compress |
| 22 | `Sounds/success.wav` | Large | 186.7 KB — could compress |

---

## Scripts (AHK)

### 🟠 HIGH

| # | File | Line(s) | Type | Description |
|---|------|---------|------|-------------|
| 23 | `master_script.ahk` | ALL | Dead Script | 11-line stub — does nothing, never referenced |
| 24 | `macro_bindings.ahk` | ALL | Dead Script | 12-line stub — does nothing, never referenced |
| 25 | `test_runner.ahk` | ALL | Dead Script | 14-line stub — `ExitApp` on line 12, immediately terminates |
| 26 | `MasterScript/config.json` | 60-76 | Config Mismatch | `DynamicRemap` and `SmartLaunch` defined but never handled in MasterScript.ahk |

### 🟡 MEDIUM

| # | File | Line(s) | Type | Description |
|---|------|---------|------|-------------|
| 27 | `MasterScript/MasterScript.ahk` | 17-19 | Missing Error | `LoadConfig()` silently returns if config missing — no user feedback |
| 28 | `MasterScript/MasterScript.ahk` | 53, 141 | Debug | Writes to `error_log.txt` and `engine_error.log` — no rotation, unbounded |
| 29 | `MasterScript/MasterScript.ahk` | 102-103 | Missing Error | Bare `catch { return }` — swallows all exceptions silently |
| 30 | `MasterScript/config.json` | 74 | Hardcoded Path | `C:\Windows\System32\notepad.exe` — hardcoded system path |

---

## Labels Key

- 🔴 **CRITICAL** — Must remove before packaging
- 🟠 **HIGH** — Should fix soon
- 🟡 **MEDIUM** — Clean up pass
- 🟢 **LOW** — Technical debt

---

**Last Updated:** August 1, 2026
