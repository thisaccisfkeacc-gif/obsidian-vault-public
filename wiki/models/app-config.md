---
tags: [model, config, settings, hotkeys]
date: 2026-08-01
sources:
  - Models/AppConfig.cs
  - ConfigManager.cs
status: current
---

# AppConfig Model 📋

`AppConfig.cs` defines the data models for application settings and hotkey bindings. These are serialized to `config.json` via `ConfigManager`.

## Core Classes

### `AppConfig`

The root configuration object loaded/saved by `ConfigManager`:

```csharp
public class AppConfig
{
    public int Version { get; set; } = 2;
    public List<AppProfile> Profiles { get; set; }
    public string SearchEngine { get; set; } = "FindText v9.8";
    public SettingsModel Settings { get; set; }
    public List<ActionItem> Hotkeys { get; set; }
    public List<SnippetItem> Snippets { get; set; }
}
```

### `SettingsModel`

Contains 100+ global toggle settings. Key properties (defaults from code):

**Recording & Capture**

| Property | Type | Default | Purpose |
|----------|------|---------|---------|
| `CaptureMousePosition` | bool | true | Capture mouse position during recording |
| `CaptureKeyboardInput` | bool | true | Capture keyboard input during recording |
| `UseAsyncMacroRecording` | bool | true | Async (non-blocking) recording |
| `AutoInsertEventDelays` | bool | true | Auto-insert delays between events |
| `TraceCaptureMode` | int | 0 | 0 = Off, 2 = All Movement |
| `MousePhysicsProfile` | int | 0 | 0 = Smooth, 1 = Instant, 2 = Original |
| `MouseMovementStyle` | string | "Smart" | "Smart" or "AlwaysAnimate" |
| `EnableSmartBundling` | bool | true | Bundle keyboard steps into clean blocks |
| `EnableModifierWrapDetection` | bool | true | Detect modifier wrap-around during recording |
| `AggressiveScrollBundling` | bool | true | Bundle consecutive scrolls |
| `HoldDelayPreset` | string | "Normal" | Hold duration preset |
| `InstantRecordMode` | bool | false | Instant record mode |
| `CaptureWindowSwitches` | bool | true | Capture window switches |
| `CaptureWindowMoves` | bool | true | Capture window moves |
| `AutoCaptureWindowSize` | bool | true | Auto-capture window dimensions |
| `SmartWindowActions` | bool | true | Smart window action detection |
| `WindowSwitchSensitivity` | string | "Normal" | Window switch sensitivity preset |
| `MinimizeToTrayDuringRecording` | bool | true | Hide to tray during recording |
| `ShowAccentBars` | bool | true | Show accent bars in editor |

**App Behavior**

| Property | Type | Default | Purpose |
|----------|------|---------|---------|
| `MinimizeToSystemTray` | bool | true | Close → tray instead of exit |
| `MinimizeAppOnPreview` | bool | true | Minimize app when previewing |
| `FastPasteTextMode` | bool | false | Default Paste mode for new Type Text blocks |
| `AppTheme` | string | "Dark" | "Dark" (default) or "Light" |
| `ShowFloatingStopButton` | bool | true | Show floating stop button |
| `AdminModeVIP` | bool | false | Run with admin elevation |
| `AutoStartEngineOnLaunch` | bool | false | Start AHK engine on launch |
| `AutoReloadOnChange` | bool | true | Recompile on config changes |
| `EnableTurboEngineMode` | bool | false | Smart priority boost while macros run |
| `EnableAdvancedMouseTriggers` | bool | false | Middle/Side click, combos |
| `MasterKillSwitchKey` | string | "Shift + Escape" | Emergency stop hotkey |
| `KillSwitchNotification` | bool | true | Notify when kill switch fires |
| `SafetyConfirmations` | bool | true | Confirm destructive actions |
| `PostTestDelay` | bool | true | Delay after testing steps |
| `CancelModeIndex` | int | 0 | Cancel mode for running macros |
| `IsLibraryModeEnabled` | bool | true | Macro library mode |
| `IsAIAssistantFeatureEnabled` | bool | false | AI assistant feature flag |
| `ShowBlockInputGuide` | bool | true | Show block input guide |
| `AutoRestoreDefaults` | bool | false | Respawn default action cards |
| `AutoStopWhenNoActionsEnabled` | bool | true | Stop engine when no actions enabled |
| `AutoCreateVirtualDesktop` | bool | false | Auto-create virtual desktop |
| `EnableWindowPickerHotkey` | bool | true | Window picker hotkey enabled |
| `AlwaysOnTopBorderEnabled` | bool | true | Always-on-top border |
| `AlwaysOnTopBorderColor` | string | "#99A855F7" | Border color |
| `AlwaysOnTopBorderThickness` | int | 1 | Border thickness |
| `UseAppNameFirst` | bool | true | Window title format |

**Preview & Humanization**

| Property | Type | Default | Purpose |
|----------|------|---------|---------|
| `FastScrollPreview` | bool | false | Skip delays between scrolls in preview |
| `FastClickPreview` | bool | false | Skip delays between clicks in preview |
| `FastTypingPreview` | bool | false | Skip delays between keystrokes in preview |
| `DefaultHumanizationLevel` | int | 2 | 0=Off, 1=Safe, 2=Normal, 3=Risky, 4=Chaos |
| `DragGraceZoneDistance` | int | 10 | Drag grace zone in px |
| `MouseDoubleClickBundlingTime` | int | 500 | Double-click bundle threshold (ms) |
| `UndoHistoryLimit` | int | 50 | Undo stack size |

**Search & Capture Visuals**

| Property | Type | Default | Purpose |
|----------|------|---------|---------|
| `SmartImageSearch` | bool | true | Smart box image search |
| `WindowSearchFallback` | bool | true | 2-tier fallback: small area → full window |
| `DefaultSearchEngine` | string | "Fast (Lightning Fast)" | Default search engine |
| `MouseAutoBindTarget` | bool | false | Auto-select image/pixel as mouse target |
| `AutoLaunchMissingWindows` | bool | false | Auto-launch missing target windows |
| `UseLastKnownPositionCache` | bool | true | Search last known find spot first |
| `LastKnownCacheThreshold` | int | 2 | Confirmations before caching |
| `AutoFlowImageCapture` | bool | true | Auto-flow image capture |
| `EnableCaptureSizeLock` | bool | true | Lock capture size |
| `IsSmartMode` | bool | true | Smart View ON by default |
| `IsStepFilterActive` | bool | false | Filter timeline by step type |
| `PreCaptureDelay` | int | 0 | Delay before capture (ms) |
| `SmartBoxSize` | int | 150 | Smart box size |
| `SmartBoxBorderThickness` | int | 1 | Smart box border |
| `PinnedBoxColor` | string | "#A855F7" | Pinned capture box color |
| `FloatingBoxColor` | string | "#FF453A" | Floating box color |
| `WarningBoxColor` | string | "#FFA500" | Warning box color |
| `CaptureCrosshairColor` | string | "#A855F7" | Crosshair color |
| `WindowHighlightColor` | string | "#10B981" | Window highlight color |
| `MagnifierStyle` | string | "Solid" | "Solid" or "Dashed" |
| `MagnifierShape` | string | "Square" | Magnifier shape |
| `MagnifierZoomLevel` | double | 5.0 | Zoom level |
| `Browsers` | Dictionary<string,bool> | chrome/msedge/brave/firefox = true | Supported browsers |

**Remote, Updates & Subscription**

| Property | Type | Default | Purpose |
|----------|------|---------|---------|
| `RemoteServerEnabled` | bool | false | Enable HTTP remote server |
| `RemoteServerPin` | string | "0000" | Mobile auth PIN (auto-random 4-digit if empty) |
| `RemoteServerPort` | int | 8745 | Server port |
| `RemoteAccessEnabled` | bool | false | Remote access (JsonIgnore) |
| `AutoCheckForUpdates` | bool | true | Auto-update checks |
| `ActiveAIProvider` | string | "OpenRouter" | AI provider |
| `EnableExperimentalAutoTrigger` | bool | true | Experimental AI auto-trigger |
| `TotalMacrosGeneratedByAI` | int | 0 | AI-generated macro counter |
| `CachedSubscriptionStatus` | string | "trial" | Offline subscription cache |
| `CachedSubscriptionEnds` | DateTimeOffset? | UtcNow + 3 days | Cached expiry |

**Snippets & Editor**

| Property | Type | Default | Purpose |
|----------|------|---------|---------|
| `SnippetsGlobalSoundEnabled` | bool | true | Sound when snippet matched |
| `SnippetsCaseInsensitive` | bool | false | Case-insensitive snippet triggers |
| `EditorShortcuts` | Dictionary<string,string> | null | Custom editor key bindings (null = defaults) |
| `ShowFavoritesInTrayMenu` | bool | true | Favorites in tray menu |
| `ActionMenuBasicCollapsed` | bool | false | Add-action menu section states |
| `ActionMenuLogicCollapsed` | bool | false | ^ |
| `ActionMenuSystemCollapsed` | bool | false | ^ |
| `ActionMenuTemplatesCollapsed` | bool | false | ^ |
| `EditorViewSectionOpen` | bool | true | Editor settings popup sections |
| `EditorRecordingSectionOpen` | bool | false | ^ |
| `EditorPlaybackSectionOpen` | bool | false | ^ |

**Profiles, Stats & Migration Flags**

| Property | Type | Default | Purpose |
|----------|------|---------|---------|
| `CustomProfiles` | List<string> | [] | Custom profile names |
| `ProfileIcons` | Dictionary<string,string> | {} | Profile → icon map |
| `LastActiveProfile` | string | "CustomActions" | Last active profile |
| `TotalMacrosExecuted` | int | 0 | Lifetime execution counter |
| `TotalSnippetsExpanded` | int | 0 | Lifetime snippet counter |
| `AppLaunchCount` | int | 0 | Launch counter |
| `UnlockedEasterEggs` | List<string> | [] | Unlocked easter eggs |
| `MacrosSeeded` / `SnippetsSeeded` / `HasAddedStarterMacros` / `HasEnabledAllByDefault` / `HasMigrated*` / `HasSeen*Guide` | bool | false | One-time migration & guide flags |

### `ActionItem`

Represents a single hotkey binding — the bridge between a macro and its trigger:

| Property | Type | Default | Purpose |
|----------|------|---------|---------|
| `Id` | string | new Guid string | Unique identifier (string!) |
| `AssignedProfile` | string | "MacroBindings" | Profile this binding belongs to |
| `Name` | string | null | Display name |
| `Description` | string | null | Description |
| `Icon` | string | ⚡ | Emoji icon (fallback ⚡) |
| `Category` | string | null | "Custom Macros", "File Launchers", etc. |
| `Function` | string | null | "Macro", "Launcher", "AppBound", etc. |
| `Key` | string | null | Hotkey string (e.g., "Ctrl+Shift+F1") |
| `Path` | string | "" | Macro ID (Guid) or file path |
| `Enabled` | bool | false | Whether the binding is active |
| `IsRemovable` | bool | true | User can delete the card |
| `TriggerMode` | TriggerMode | Single | How the hotkey activates |
| `TriggerClickCount` | int | 2 | Clicks for DoubleTap |
| `TriggerDuration` | int | 500 | Hold duration (ms) |
| `TriggerPreset` | string | "Normal" | DoubleTap/LongPress preset |
| `TogglePath`/`TogglePath2-4` | string | null | Toggle slot macro IDs |
| `ToggleSlotCount` | int | 2 | Toggle slots (clamped 2–5) |
| `ToggleTimeoutMs` | int | 2000 | Toggle window (ms) |
| `ToggleShowState` | bool | false | Show tooltip on toggle |
| `ShowTriggerFeedback` | bool | false | Tooltip feedback on trigger |
| `ScopeMode` | AppScopeMode | Global | Global / Include / Exclude |
| `TargetApp` | string | "" | Comma-separated app names |
| `WindowAction` | string | "Activate" | App-Bound window action |
| `TargetWindowX/Y/Width/Height` | int | -1 | Captured window bounds |
| `ScheduleIntervalSeconds` | int | 60 | Schedule trigger interval |
| `ScheduleRunOnStart` | bool | false | Run on start |
| `ScheduledTime` | string | null | "HH:mm" 24-hour |
| `ScheduledDays` | string | "Daily" | "Daily"/"Weekdays"/"Mon",... |
| `ScreenEventDetectMode` | string | "Image" | "Image" / "Pixel" / "UIElement" |
| `TriggerImage` | string | null | Screen-event image path |
| `TriggerPixelHex`/`TriggerPixelX`/`TriggerPixelY` | — | null / 0 | Pixel target |
| `ScreenEventTolerance` | double | 0.025 | Match tolerance |
| `ScreenEventSearchBoxSize` | int | 60 | Search box size |
| `ScreenEventNotifyOnFound` | bool | false | Notify when auto-detect finds target |
| `ScreenEventPollInterval` | int | 500 | Poll interval (ms, min 100) |
| `ScreenEventImageX/Y/Width/Height` | int | 0 | Captured region |
| `ScreenEventSearchScope` | string | null | Search scope string |
| `ScreenEventFireMode` | string | "OnAppear" | "OnAppear"/"WhileVisible"/"Cooldown" |
| `ScreenEventCooldownMs` | int | 5000 | Cooldown (ms) |
| `ScreenEventFireLimit` | int | 0 | Fire limit (0 = unlimited) |
| `ScreenEventNotifyTooltip` | bool | true | Tooltip on trigger |
| `ScreenEventNotifySound` | bool | false | Sound on trigger |
| `ScreenEventUI*` | string | null | UI-element match fields |
| `TextExpansionValue` | string | null | Text expander payload |

### `TriggerMode` Enum

11 values (from `PowerX.Core\Models\AppEnums.cs`):

```
Single, DoubleTap, Hold, Release, LongPress, Toggle, ScreenEvent, Schedule, MobileRemote, PressAndRelease, ScheduledTime
```

## ConfigManager

Static class (`PowerX.Services\Services\ConfigManager.cs`) that handles JSON serialization:

- `ConfigManager.Current` — the loaded `AppConfig` instance
- `ConfigManager.Initialize()` — creates the folder if missing, then loads config from disk (with `.bak` restore on corruption); on first run scaffolds default profiles and hotkeys
- `ConfigManager.Save()` — writes to disk (debounced 500ms)
- `ConfigManager.ForceSave()` — writes immediately
- `ConfigManager.ConfigUpdated` event — fired after save, triggers engine reload

### Storage Path
```
%LOCALAPPDATA%/PowerXKeys/config.json
```
(defined as `ConfigPath` in ConfigManager.cs; a `.bak` backup is kept alongside)

## Key Files

- [AppConfig.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Core/Models/AppConfig.cs)
- [ConfigManager.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Services/Services/ConfigManager.cs)

## Related Pages

- [[settings-dashboard]]
- [[script-library]]
- [[execution-pipeline]]
- [[macro-item]]
