# PowerX Keys V2 — Audit Tracker

**Last Updated:** 2026-08-01  
**Total Components:** 102  
**Scanned:** 102/102 (100%)  
**Remaining:** 0 — ALL SCANNED

---

## Master Progress

### Blocks (21/21 — DONE)

| # | Block | Bugs | Dead Code | Inconsistencies | Priority Fixes | Status |
|---|-------|------|-----------|-----------------|----------------|--------|
| 1 | UIElement | 1 | 0 | 0 | 1 (proximity) | ✅ Fixed |
| 2 | ImageSearch | 1 | 0 | 1 | 0 | ✅ Done |
| 3 | PixelSearch | 1 | 0 | 0 | 1 (double-call) | ✅ Fixed |
| 4 | WindowAction | 1 | 1 | 0 | 1 (AHK v1 syntax) | ✅ Fixed |
| 5 | LogicIf | 1 | 0 | 0 | 1 (WPF binding) | ✅ Fixed |
| 6 | WaitUntil | 0 | 0 | 0 | 0 | ✅ Clean |
| 7 | MouseClick | 0 | 0 | 0 | 0 | ✅ By Design |
| 8 | MouseTrace | 1 | 0 | 0 | 1 (right-drag) | ✅ Fixed |
| 9 | Keyboard | 1 | 0 | 0 | 1 (SafeSendKeys) | ✅ Fixed |
| 10 | Text | 0 | 0 | 1 | 0 | ✅ Done |
| 11 | Delay | 0 | 0 | 0 | 0 | ✅ Clean |
| 12 | LoopSequence | 0 | 1 | 0 | 0 | ✅ Done |
| 13 | GroupHeader | 0 | 0 | 0 | 0 | ✅ Clean |
| 14 | Popup | 0 | 0 | 0 | 0 | ✅ By Design |
| 15 | Notification | 0 | 0 | 0 | 1 (AHK v1 syntax) | ✅ Fixed |
| 16 | SystemSound | 0 | 0 | 0 | 0 | ✅ Clean |
| 17 | UserInput | 0 | 0 | 0 | 0 | ✅ Clean |
| 18 | WaitForKey | 0 | 0 | 0 | 0 | ✅ By Design |
| 19 | FileLauncher | 0 | 0 | 0 | 0 | ✅ Clean |
| 20 | CallMacro | 1 | 0 | 0 | 1 (ID-vs-Name) | ✅ Fixed |
| 21 | SetVariable | 0 | 0 | 0 | 1 (escaping) | ✅ Fixed |

### Systems (4/4 — DONE)

| # | System | Bugs | Dead Code | Inconsistencies | Priority Fixes | Status |
|---|--------|------|-----------|-----------------|----------------|--------|
| 22 | Recording | 2 | 1 | 0 | 2 (deadlock, EndX/EndY) | ⏳ Pending |
| 23 | Capture | 2 | 2 | 0 | 1 (DPI) | ⏳ Pending |
| 24 | Undo/Redo | 2 | 1 | 0 | 2 (Timer leak, StepsEqual) | ⏳ Pending |
| 25 | Hotkey | 1 | 2 | 0 | 1 (RWin guard) | ⏳ Pending |

### Pipeline (1/1 — DONE)

| # | Component | Bugs | Dead Code | Inconsistencies | Priority Fixes | Status |
|---|-----------|------|-----------|-----------------|----------------|--------|
| 26 | AHK Pipeline | 2 | 1 | 1 | 2 (token race, ErrorStdOut) | ✅ Fixed |

### Remaining (20/27 — DONE)

| # | Component | Type | Risk | Status | Bugs | Security | Dead Code | Notes |
|---|-----------|------|------|--------|------|----------|-----------|-------|
| 27 | ConfigManager | Service | High | ✅ Done | 7 | 4 | 3 | Race conditions, plaintext PIN |
| 28 | MacroDatabase | Manager | High | ✅ Done | 7 | 0 | 2 | Tolerance mutation (fixed), Window Library connection |
| 29 | RemoteServerService | Service | High | ✅ Done | 5 | 10 | 1 | No TLS, hardcoded PIN, no token expiry |
| 30 | AutoUpdateService | Service | High | ✅ Done | 7 | 6 | 2 | Zero integrity verification, non-functional URL |
| 31 | SmartView | ViewModel | Medium | ✅ Done | 5 | 0 | 3 | PropertyChanged leak, stale comments |
| 32 | Theme | Service | Low | ✅ Done | 15 | 0 | 3 | 10+ hardcoded dark-mode colors, GhostButton shadowed |
| 33 | Config/Settings | ViewModel | High | ✅ Done | 6 | 0 | 4 | ResetConfigSelective mismatch, orphaned properties |
| 34 | DebugLogger | Service | Low | ✅ Done | 5 | 0 | 0 | Double timestamps, silent data loss, AHK write conflict |
| 35 | NetworkTimeService | Service | Low | ✅ Verified | — | — | — | File EXISTS — PowerX.Services\Services\NetworkTimeService.cs (row resolved) |
| 36 | EasterEggService | Service | Low | ✅ Done | 3 | 0 | 1 | Config save no error handling, dead IsUnlocked() |
| 37 | TelemetryService | Service | Low | ✅ Done | 3 | 4 | 1 | Non-functional (empty credentials), PII leakage |
| 38 | AlwaysOnTopOverlayService | Service | Low | ✅ Done | 5 | 0 | 1 | Double-positioning flicker, blanket catch { } |
| 39 | SystemActionService | Service | Medium | ✅ Resolved | 0 | 0 | 0 | Class DELETED from codebase — zero references remain (row resolved) |
| 40 | ShortcutManager | Service | Medium | ✅ Done | 3 | 0 | 0 | Rebind silent drop, LoadFromConfig swallows all |
| 41 | FindTextService | Service | Low | ✅ Done | 3 | 0 | 0 | Null bitmap crash, not OCR (pixel-pattern encoder) |
| 42 | StopService | Service | Medium | ✅ Done | 3 | 0 | 1 | Overlays not unpinned, redundant logging |
| 43 | PerformanceProxy | Service | Low | ✅ Resolved | 0 | 0 | 0 | Class removed — zero references, build fine (row resolved) |
| 44 | MacroTransferManager | Manager | Medium | ✅ Done | 6 | 3 | 2 | Dead hotkey dedup loops, no ZIP bomb protection |
| 45 | ScriptManager | Manager | Medium | ✅ Done | 7 | 0 | 3 | Stderr read on disposed process, dead WaitForFileReady |
| 46 | AILoginManager | Manager | Low | ✅ Resolved | 0 | 0 | 0 | Class DELETED from codebase — zero references remain (row resolved) |
| 47 | SettingsViewModel | ViewModel | Medium | ✅ Done | 6 | 0 | 6 | Missing bindings, orphaned properties (AdminModeVIP IS persisted — plain serialized SettingsModel property, AppConfig.cs:70) |
| 48 | AuthViewModel | ViewModel | Medium | ✅ Done | 1 | 1 | 0 | IsBusy stuck on browser-open failure |
| 49 | AIAssistantViewModel | ViewModel | Low | ✅ Done | 4 | 0 | 2 | Non-functional (empty PROXY_URL), Dispatcher.Invoke blocking |
| 50 | TextSnippetsViewModel | ViewModel | Low | ✅ Done | 5 | 0 | 2 | Config save skipped on conflict, timer leak |
| 51 | MainViewModel | ViewModel | Medium | ✅ Done | 10 | 0 | 4 | Race on _isEngineRunning, dispatcher.Invoke blocking |
| 52 | CoreData | Model | Medium | ✅ Done | 9 | 0 | 5 | 6 properties not persisted, single MacroItem.cs in PowerX.Core\Models (duplicate row resolved) |
| 53 | Converters | Converter | Low | ✅ Done | 5 | 0 | 5 | 5 dead converters, mutable brushes not frozen |

### Core Components (8/9 — DONE)

| # | Component | Type | Status | Bugs | Dead Code | Notes |
|---|-----------|------|--------|------|-----------|-------|
| 54 | MacroEditorViewModel.Core | ViewModel | ✅ Done | 6 | 2 | Dead CleanupStepFiles, swallowed exception, VirtualSourceSteps mutation |
| 55 | MacroEditorViewModel.Recording | ViewModel | ✅ Done | 3 | 1 | Widget fires before recording starts, _isRecordingActive not reset |
| 56 | MacroEditorViewModel.Capture | ViewModel | ✅ Done | 6 | 4 | Incorrect WIN_SMART title, dead needsSearchArea, commented-out code |
| 57 | MacroEditorView.Events | View | ✅ Done | 5 | 4 | Drag-drop event routing inconsistency, async re-entrance, dead handlers |
| 58 | MacroEditorView.DragDrop | View | ✅ Done | 5 | 3 | Branch drops ignore cursor, stale _lastDropElement, index miscalculation |
| 59 | ProfileCreationWindow | View | ✅ Done | 4 | 1 | Custom emoji stale precedence, inconsistent reserved-name validation |
| 60 | ActionModels + AppEnums | Model | ✅ Done | 3 | 2 | ActionType.KeystrokeAction unhandled, namespace split |
| 61 | Helpers (VisualTreeExtensions) | Helper | ✅ Done | 1 | 0 | Redundant fallback at 20+ call sites |

### UI Components (Batch 2 — DONE)

| # | Component | Type | Status | Bugs | Dead Code | Notes |
|---|-----------|------|--------|------|-----------|-------|
| 62 | CrashReportWindow | Dialog | ✅ Done | 0 | 0 | Clean |
| 63 | ForceUpdateWindow | Dialog | ✅ Done | 1 | 0 | No-op Closing unsubscribe, no try-catch on Process.Start |
| 64 | HardwareLockWarningDialog | Dialog | ✅ Done | 0 | 0 | Clean |
| 65 | SubscriptionExpiredWindow | Dialog | ✅ Done | 0 | 0 | Clean |
| 66 | SafetyWarningDialog | Dialog | ✅ Done | 0 | 0 | Clean |
| 67 | NamingConflictDialog | Dialog | ✅ Done | 0 | 0 | Clean |
| 68 | DarkMessageBoxWindow | Dialog | ✅ Done | 2 | 1 | DialogResult not set for keyboard shortcuts |
| 69 | InputPromptWindow | Dialog | ✅ Done | 1 | 0 | BrushFromHex no hex validation |
| 70 | DropdownPromptWindow | Dialog | ✅ Done | 0 | 1 | Duplicate Close_Click/Cancel_Click |
| 71 | ExportMacroPickerDialog | Dialog | ✅ Done | 2 | 1 | Dead Macro property, wrong selected count |
| 72 | ImageStudioWindow | Editor | ✅ Done | 2 | 3 | DialogResult in non-ShowDialog, heavy RefreshBinarization |
| 73 | CaptureOverlay | Editor | ✅ Done | 3 | 4 | Thread.Sleep on UI thread, empty catch blocks, perf on mouse move |
| 74 | CaptureLibraryWindow | Editor | ✅ Done | 1 | 1 | RefreshData loads all 4 types every time |
| 75 | CoordinatePickerWindow | Editor | ✅ Done | 0 | 0 | Clean |
| 76 | CoordinateEditDialog | Editor | ✅ Done | 0 | 0 | Clean |
| 77 | OffsetCaptureWindow | Editor | ✅ Done | 1 | 1 | Silent catch in PerformLiveSearchAsync |
| 78 | TargetPickerWindow | Editor | ✅ Done | 0 | 0 | Clean |
| 79 | WindowPickerWindow | Editor | ✅ Done | 1 | 1 | Re-enumerates windows on every Activated, no Escape handler |
| 80 | KeyCaptureWindow | Editor | ✅ Done | 1 | 1 | Escape mode not truly enforced |
| 81 | MacroStepCard | UI | ✅ Done | 0 | 1 | Dead FindParent method |
| 82 | MacroStepTemplates | UI | ✅ Done | 0 | 0 | Clean |
| 83 | MacroEditorOverlays | UI | ✅ Done | 0 | 0 | Clean |
| 84 | MacroEditorCheatSheet | UI | ✅ Done | 0 | 1 | Empty Click handler |
| 85 | RecordingWidgetView | UI | ✅ Done | 0 | 0 | Clean |
| 86 | TextSnippetsView | UI | ✅ Done | 0 | 3 | Orphaned ShowChoiceDropdownDialog |
| 87 | ScriptLibraryView | UI | ✅ Done | 2 | 3 | Duplicate right-click block, NotImplementedException in ConvertBack |
| 88 | SettingsDashboardView | UI | ✅ Done | 0 | 0 | Clean |
| 89 | SkeletonOverlay | UI | ✅ Done | 0 | 0 | Clean |
| 90 | SplashWindow | UI | ✅ Done | 0 | 0 | Clean |
| 91 | MagnifierPreview | UI | ✅ Done | 0 | 0 | Clean |
| 92 | MousePathOverlayWindow | UI | ✅ Done | 0 | 1 | 7 unused P/Invoke declarations |
| 93 | UIElementHighlightWindow | UI | ✅ Done | 0 | 0 | Clean |
| 94 | ToggleCommandBehavior | UI | ✅ Done | 0 | 0 | Clean |
| 95 | CustomActionCard | UI | ✅ Done | 0 | 0 | Clean |
| 96 | ImagePreviewWindow | UI | ✅ Done | 0 | 0 | Clean |
| 97 | AIAssistantView | UI | ✅ Done | 0 | 0 | Clean |
| 98 | AuthWindow | UI | ✅ Done | 0 | 0 | Clean |

### App Core (4/4 — DONE)

| # | Component | Type | Status | Bugs | Security | Dead Code | Notes |
|---|-----------|------|--------|------|----------|-----------|-------|
| 99 | App.xaml.cs | Entry | ✅ Done | 7 | 0 | 0 | Null mutex dereference, _isExiting not volatile, ConfigManager null risk |
| 100 | MainWindow.xaml.cs | Window | ✅ Done | 7 | 0 | 1 | Silent catch blocks, no ScriptManager error handling, dead overlay timer |
| 101 | FileAssociationService | Service | ✅ Done | 4 | 4 | 0 | Overwrites existing associations, argument injection risk |
| 102 | PowerX_Updater/Program.cs | Updater | ✅ Done | 4 | 3 | 0 | No download integrity verification (HIGH), HTTP allowed, filename mismatch |

---

## Priority Fixes Tracker

### From Blocks/Systems/Pipeline (Earlier)

| # | Fix | Source | Status | Assignee |
|---|-----|--------|--------|----------|
| 1 | CallMacro ID-vs-Name | Block | ✅ Fixed | Other agent |
| 2 | Keyboard SafeSendKeys | Block | ✅ Fixed | Other agent |
| 3 | Recording deadlock | System | ⏳ Pending | Other agent |
| 4 | Recording trace-off EndX/EndY | System | ⏳ Pending | Other agent |
| 5 | Capture DPI double-scaling | System | ⏳ Pending | Other agent |
| 6 | Undo/Redo Timer.Tick leak | System | ⏳ Pending | Other agent |
| 7 | Undo/Redo StepsEqual incomplete | System | ⏳ Pending | Other agent |
| 8 | Hotkey RWin guard | System | ⏳ Pending | Other agent |
| 9 | ImageSearch extra closing paren | Pipeline | ❌ False Positive | No extra paren in actual code |
| 10 | Double GlobalMacroToken increment | Pipeline | ✅ Fixed | ScriptCompilerService.cs |
| 11 | `#ErrorStdOut` missing `"**"` | Pipeline | ✅ Fixed | SingleStep.cs |

### From New Scans (HIGH PRIORITY)

| # | Fix | Source | Status | Assignee |
|---|-----|--------|--------|----------|
| 12 | RemoteServerService: No TLS — plaintext HTTP | Security | ⏳ Pending | Other agent |
| 13 | RemoteServerService: Hardcoded PIN "1234" | Security | ⏳ Pending | Other agent |
| 14 | RemoteServerService: No session token expiry | Security | ⏳ Pending | Other agent |
| 15 | RemoteServerService: Timing-safe PIN comparison | Security | ⏳ Pending | Other agent |
| 16 | AutoUpdateService: Zero integrity verification on downloads | Security | ⏳ Pending | Other agent |
| 17 | AutoUpdateService: New HttpClient per call (socket exhaustion) | Bug | ⏳ Pending | Other agent |
| 18 | ConfigManager: Dispatcher.Invoke no timeout (deadlock risk) | Bug | ⏳ Pending | Other agent |
| 19 | MacroDatabase: Window Library bypasses shared connection | Bug | ⏳ Pending | Other agent |
| 20 | SettingsViewModel: ResetConfigSelective parameter mismatch | Bug | ⏳ Pending | Other agent |
| 21 | Theme: GhostButton style defined twice (shadowing) | Bug | ⏳ Pending | Other agent |
| 22 | Theme: 10+ hardcoded dark-mode colors in App.xaml | Bug | ⏳ Pending | Other agent |
| 23 | CoreData: 6 properties not persisted (HoldDelayMs, ReleaseDelayMs, SourceWindowX/Y, FindAllMatches, MatchSelectMode, DefaultTargetSize) | Bug | ⏳ Pending | Other agent |
| 24 | CoreData: Duplicate MacroItem.cs files (2 assemblies) | Bug | ✅ Fixed | Single file remains — PowerX.Core\Models\MacroItem.cs |
| 25 | AIAssistantViewModel: Non-functional (empty PROXY_URL) | Bug | ⏳ Pending | Other agent |
| 26 | AIAssistantViewModel: Dispatcher.Invoke blocking UI thread | Bug | ⏳ Pending | Other agent |
| 27 | MainViewModel: Race on _isEngineRunning | Bug | ⏳ Pending | Other agent |
| 28 | MainViewModel: Dispatcher.Invoke blocking UI thread | Bug | ⏳ Pending | Other agent |
| 29 | PerformanceProxy: MISSING FILE — build-breaking | Bug | ✅ Fixed | Class removed — zero references, build fine |
| 30 | MacroTransferManager: No ZIP bomb protection | Security | ⏳ Pending | Other agent |
| 31 | MacroTransferManager: Dead hotkey dedup loops | Bug | ⏳ Pending | Other agent |
| 32 | Converters: 5 dead converters | Dead Code | ⏳ Pending | Other agent |
| 33 | SystemActionService: ENTIRE CLASS IS DEAD CODE | Dead Code | ✅ Fixed | Class deleted — zero references |
| 34 | AILoginManager: ENTIRE CLASS IS DEAD CODE | Dead Code | ✅ Fixed | Class deleted — zero references |

**Fixed (all sessions):** 14  
**False flags:** 2  
**Pending:** 28

---

## Summary

- **Total findings:** 340+ (across 102 components)
- **Priority fixes:** 28 pending, 14 fixed, 2 false flags
- **Security issues:** 32+ (RemoteServerService, AutoUpdateService, ConfigManager, TelemetryService, MacroTransferManager, PowerX_Updater, FileAssociationService)
- **Dead code:** 0 entire classes (SystemActionService + AILoginManager deleted) + 5 converters + 35+ orphaned methods/handlers + 7 unused P/Invoke declarations
- **Build issues:** 4 semicolons in ConfigManager (fixed), PowerX.Services missing type references (resolved — PerformanceProxy removed, build fine)
- **Scanned:** 102/102 (100%)
- **Remaining:** 0 — ALL COMPONENTS SCANNED
