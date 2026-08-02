---
tags: [model, macro, steps, data-structure]
date: 2026-08-01
sources:
  - Models/MacroItem.cs
status: current
---

# MacroItem Model 🧩

`MacroItem.cs` defines the core data structures for macros and their individual steps. These are the primary objects stored in SQLite and manipulated by the macro editor.

## MacroItem

Represents a complete macro — a named, reusable automation sequence.

| Property | Type | Default | Purpose |
|----------|------|---------|---------|
| `Id` | Guid | auto-generated | Unique identifier |
| `IdString` | string | — | `Id.ToString()` helper |
| `Name` | string | null | User-visible name |
| `Icon` | string | null | Emoji icon |
| `AssignedProfile` | string | "MacroBindings" | Profile this macro belongs to |
| `MacroSteps` | `ObservableCollection<MacroStep>` | [] | Ordered list of actions |
| `IsFavorite` | bool | false | Favorite flag |
| `IsAssigned` | bool | false | Bound to a hotkey |
| `PlaybackSpeed` | double | 1.0 | Speed multiplier (lives on the macro!) |
| `MousePhysicsProfile` | int | 0 | 0 = Smooth, 1 = Raw (lives on the macro!) |
| `TraceCaptureMode` | int | 0 | 0 = Off, 2 = Full Continuous Trace |
| `BlockHardwareInput` | bool | false | Block real input during run |
| `AutoDelayEnabled` | bool | false | Smart auto-delay |
| `AutoDelayPreset` | string | "short" | "short" / "medium" / "long" / "custom" |
| `AutoDelayMs` | int | 100 | Auto-delay ms (10–5000) |
| `IsHumanized` | bool | false | Humanized timing |
| `DefaultHumanizationLevel` | int | 2 | Global default level |
| `TriggerKey` / `TriggerModeString` / `FastSteps` | — | null | Experimental AI auto-trigger fields |

No `CreatedAt` / `ModifiedAt` — these are not stored.

## MacroStep

Represents a single action within a macro. This is the workhorse data model — virtually every feature touches this class.

### Core Properties

| Property | Type | Default | Purpose |
|----------|------|---------|---------|
| `Id` | Guid | auto-generated | Unique step identifier |
| `Type` | MacroStepType | — | What kind of action (NOT `StepType`) |
| `Value` | string | null | Core payload — key combo ("Ctrl+C"), text to type, sound name, launch path, etc. (NOT `KeyString`/`TextToType`) |
| `StepName` | string | null | Display name (falls back to friendly per-type name) |
| `X`, `Y` | double? | null | Target coordinates |
| `EndX`, `EndY` | double? | null | End coordinates (for drag) |
| `Duration` | int? | null | Wait time in ms (NOT `DelayMs`) |
| `ActionTarget` | string | "Coordinates" | "Coordinates", "FoundTarget", "Active Window", etc. |
| `CoordinateMode` | string | "Screen" | "Screen" or "Window" coords |
| `KeyActionType` | string | "Press" | "Press", "Hold Down", "Released Up" |
| `ScrollAmount` | int? | 1 | Scroll lines |
| `ClickCount` | int | 1 | Click count / loop iterations |
| `IsDisabled` | bool | false | Step skipped (inverted: old `IsEnabled` gone) |
| `IsManuallyAdded` | bool | false | Prevents smart cleanup from removing |
| `IsSourceDisabled` | bool | false | Logic source disabled |

### Visual Search Properties

| Property | Type | Default | Purpose |
|----------|------|---------|---------|
| `SearchImageFilename` | string | null | Image file reference |
| `FindTextCode` | string | null | FindText pattern string |
| `FindTextTolerance` | double | 0.01 | FindText tolerance (1% Fine) |
| `FindTextBgTolerance` | double | 0.0 | Background tolerance (err0) |
| `UseFastEngine` | bool | true | Use FindText engine |
| `SearchEngine` | string | "Fast (Find Text Tool)" | Engine name |
| `SearchScopeSummary` | string | null | "SMART_BOX", "Full Screen", `WIN_LIVE:`, `WIN_REL:`, `WIN_SMART:`, `WIN:`, or `x,y,x2,y2` |
| `SearchWidth`, `SearchHeight` | double? | null | Search box size |
| `SmartSearchBoxSize` | int | 60 | Smart box size |
| `TargetColorHex` | string | null | Pixel color target |
| `Tolerance` | int | 3 | Pixel tolerance |
| `ImageTolerance` | int | 0 | Image tolerance % |
| `FailIfMissing` | bool | true | Fail when not found |
| `FindAllMatches` | bool | false | Highlight every match |
| `MatchSelectMode` | string | "First" | "First" or "Last" match |
| `EnableSmartRetry` | bool | false | Smart retry on failure |
| `MaxRetries` | int | 5 | Retry count |
| `RetryDelayMs` | int | 500 | Retry delay |
| `SearchStep*` toggles | bool | false | Search cascade steps |
| `LastKnown*` | int | -1/0 | Last known position cache |

### Mouse Trace Properties

| Property | Type | Default | Purpose |
|----------|------|---------|---------|
| `TraceFileId` | string | null | Guid linking to `trace_{id}.dat` (NOT `TracePath`) |
| `TracePointCount` | int | 0 | Point count |
| `HoldDelayMs` / `ReleaseDelayMs` | int | 0 | Hold/release delays |
| `DragComplexity` | string | null | "Simple" or "Complex" (cached) |
| `IsDesktopWindow` | bool | false | Target is the Desktop |

> `PlaybackSpeed` and `MousePhysicsProfile` live on **MacroItem**, not the step.

### Logic Block Properties

| Property | Type | Default | Purpose |
|----------|------|---------|---------|
| `ChildSteps` | `ObservableCollection<MacroStep>` | [] | Children (Groups, Loops, If-true) |
| `ChildStepsFalse` | `ObservableCollection<MacroStep>` | [] | Else branch children |
| `ParentId` | Guid? | null | Parent step link |
| `IsFalseBranch` | bool | false | 1 if under Else branch |
| `GroupId` | Guid? | null | Identifier linking steps inside a Group |
| `GroupNote` | string | null | Group note |
| `GroupColor` | string | "#F5C518" | Group color |
| `LogicMode` | string | "AboveStepSuccess" | "AboveStepSuccess", "AboveStepFailed", "NamedBlockSuccess", "NamedBlockFailed", "VariableEquals", "VariableNotEquals" |
| `LogicSource` | string | null | Named source step result name |
| `ResultName` | string | null | Output result name |
| `LogicVariableName` | string | null | Variable to compare (no `LogicCondition` — mode covers it) |
| `LogicExpectedValue` | string | null | Expected comparison value |

No `LoopCount` — loops use `ClickCount` + `ChildSteps`.

### User Input Properties

| Property | Type | Default | Purpose |
|----------|------|---------|---------|
| `InputType` | string | "Text" | "Text", "Dropdown", "YesNo" |
| `InputOptions` | string | null | Comma-separated options |
| `InputVariableName` | string | null | AHK variable name for result |
| `UseVariable` | bool | false | Text block pulls from a variable |
| `VariableSource` | string | null | Source variable name |

No `InputPrompt` — the prompt text lives in `Value`.

### Sound & Popup Properties

| Property | Type | Default | Purpose |
|----------|------|---------|---------|
| `SoundType` | string | "Success" | "Beep", "Custom File", etc. |
| `SoundFilePath` | string | null | Path to custom audio file |
| `SmartWait` | bool | true | Smart wait before sound |
| `PopupMode` | string | "Checkpoint" | Popup mode |
| `PopupTimeout` | int | 3 | Auto-continue seconds |
| `NotificationIcon` | string | "Information" | Tray icon |
| `NotificationSilent` | bool | false | Silent notification |

### Window Action Properties

| Property | Type | Default | Purpose |
|----------|------|---------|---------|
| `WindowTitle` | string | null | Captured window title |
| `CaptureWindowParameters` | bool | false | Remember position/size |
| `WindowX` / `WindowY` / `WindowWidth` / `WindowHeight` | int | 0 | Captured window bounds |
| `BrowserTabSwitchEnabled` | bool | false | Toggle active browser tab switching |
| `BrowserTabName` | string | null | Active tab name captured during window capture |
| `AutoLaunchIfMissing` | bool | false | Launch app if window missing |
| `AutoLaunchPath` | string | null | App to launch |
| `SearchOtherWindows` | bool | false | Search other windows |

### Wait-For & UI Element Properties

| Property | Type | Default | Purpose |
|----------|------|---------|---------|
| `WaitConditionType` | string | "ImageFound" | "ImageFound", "PixelFound", "WindowActive" |
| `CheckIntervalMs` | int | 250 | Poll interval |
| `OnTimeoutAction` | string | "Stop" | Timeout behavior |
| `WaitContinueKey` / `WaitCancelKey` | string | "Enter" / "Escape" | WaitForKey keys |
| `WaitKeyMode` | int | 0 | 0=Specific, 1=StrictOK, 2=StrictCancel |
| `WaitMessageType` | string | "ToolTip" | "ToolTip" or "Popup" |
| `ShowWaitMessage` | bool | true | Show wait prompt |
| `UIElementName` / `UIAutomationId` / `UIClassName` / `UIControlType` | string | null | UI Automation match fields |
| `UIWindowTitle` / `UIProcessName` | string | null | Window context |
| `UIAction` | string | "Click" | "Click", "Read Text", "Set Text", "Toggle", etc. |
| `UIFindMode` | string | "Exact" | "Exact", "SameApp", "Find Latest", "Find First" |
| `UISetTextValue` | string | null | Text for "Set Text" |
| `UITimeoutSeconds` | int | 10 | Wait-until timeout |
| `UIFallbackToCoordinates` | bool | false | Fall back to X,Y |
| `UIMatchByProcess` | bool | false | Match by process name |
| `UIBackgroundMode` | bool | false | Run without focus |
| `UsePhysicalClick` | bool | false | Physical mouse vs InvokePattern |
| `UIElementPath` | string | null | Breadcrumb path |
| `UIScreenshotPath` | string | null | Preview screenshot |
| `UIScrollIntoView` | bool | false | Scroll container until visible |

## MacroStepType Enum

Numeric values map to the V1 documentation (Image=10, Pixel=11, Window=12, MouseTrace=13):

```
Keyboard = 0,  MouseClick = 3,  Delay = 7,  Text = 8,
ImageSearch = 10,  PixelSearch = 11,  WindowAction = 12,  MouseTrace = 13,
Popup = 14,  Notification = 15,  SystemSound = 16,
UserInput = 17,  WaitForKey = 18,  FileLauncher = 19,
LogicIf = 20,  LogicElse = 21,  LogicEndIf = 22,  UIElement = 23,
GroupHeader = 30,  LoopSequence = 31,
WaitUntil = 40,  CallMacro = 41,  SetVariable = 42,
InsertTemplate = 45,  LogicIfExperimental = 50
```

## GlobalStepChanged Event

`MacroStep` exposes a static `GlobalStepChanged` event. When any step property changes, it fires this event, which `MacroEditorViewModel` listens to for setting the `IsDirty` flag. `MacroItem` has its own static `GlobalStepChanged` too.

## Key Files

- [MacroItem.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Core/Models/MacroItem.cs)

## Related Pages

- [[macro-editor]]
- [[database-schema]]
- [[execution-pipeline]]
- [[image-recognition]]
- [[mouse-trace]]
