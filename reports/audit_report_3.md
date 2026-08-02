# 🔍 Audit Report 3: Macro Editor Navigation, Context Menus & Recording

**Date:** 2026-07-26  
**Project:** PowerX Keys  
**Scope:** MacroEditorView.xaml (2258 lines), MacroEditorView.xaml.cs (329 lines), MacroEditorViewModel.Commands.cs (1400 lines), MacroEditorViewModel.Core.cs (1307 lines), MacroEditorViewModel.Properties.cs, MacroEditorViewModel.Recording.cs (774 lines), MacroStepCard.xaml (949 lines)

### ✅ Final Verification — Resolved Items
| Item | Status | Note |
|------|--------|------|
| FastEngine 3rd-tier Full-Screen Fallback (`hasSmartSearchFallback`) | ✅ PASS | Verified in `ScriptCompilerService.cs:2226-2268` |
| PixelSearch search bounds protection | ✅ PASS | Defaults to 20x20 in `MacroExecutionService.cs:1625-1626` |

---

## 1. Macro Editor Layout & Navigation

### Header Toolbar (Row 0)

| Element | Lines | Verdict |
|---------|-------|---------|
| Back button | 202 | ✅ PASS — Navigates to macro library |
| AI Live Build indicator | 232-309 | ✅ PASS — Animated pulsing badge with rainbow glow, visible when `IsLiveBuildInProgress`, disabled in PerformanceMode |
| Error Badge | 309-337 | ✅ PASS — `"{0} Issue(s)"` in red, collapsed when count=0, triggers `FindError_Click` |
| Delete Macro | 338-364 | ✅ PASS — Trash icon turns red on hover, `DeleteMacroCommand`, visible when saved |
| Preview Macro | 365-395 | ✅ PASS — Blue play icon, `UnifiedPreviewCommand`, 40% opacity when disabled |
| Save Macro | 396 | ✅ PASS — Blue button, `SaveButton_Click`, disabled when `!IsSaveReady` |
| Record Button | 435-437 | ✅ PASS — Pill button, `ToggleRecordCommand` |
| Add Step ToggleButton | 444-474 | ✅ PASS — Opens action picker popup, disabled during live build |
| Undo Button | 969-996 | ✅ PASS — `UndoCommand`, bound to `CanUndo`, purple hover |
| Redo Button | 999-1025 | ✅ PASS — `RedoCommand`, bound to `CanRedo`, blue hover |
| Smart/Raw Toggle | 1030-1053 | ✅ PASS — Smart View bundling toggle, green when active |
| Hide Delays Toggle | 1055-1082 | ✅ PASS — Toggle delay visibility, blue slash when active |
| Step Filter Dropdown | 1084-1099 | ✅ PASS — Type filter combobox, visible when `IsStepFilterActive` |
| Editor Settings Gear | 1104-1284 | ✅ PASS — Popup with View & Display, How It Runs, Help & Shortcuts |

### Add Action Popup (lines 476-956)

- ✅ 3 collapsible sections: BASIC ACTIONS, LOGIC, ADVANCED
- ✅ 18+ action types (Mouse, Keyboard, Wait, Text, etc.)
- ✅ Drag-to-add via `PreviewMouseLeftButtonDown` / `MouseMove` handlers
- ✅ All use `FolderCardButtonStyle` with DynamicResource tokens
- ⚠️ **No hardcoded colors** in this section — all use `DynamicResource`

### Empty Timeline State (lines 1842-1862)

- ✅ Show when `CurrentMacro.MacroSteps.Count == 0 && !IsTimelineProcessing`
- ✅ Centered icon + "No steps yet" + "Hit Record or Add Action to get started"
- ✅ Proper use of `DynamicResource` for all colors

### Preview Button Enablement (`CanPreview`, Properties.cs:479-527)

- ✅ Requires at least one non-passive step (filters Delay, GroupHeader, etc.)
- ✅ All steps must have `IsValid == true`
- ✅ Recursive validation via `IsTimelineFullyValidated()`

### Save Button Enablement (`IsSaveReady`, Properties.cs:656)

- ✅ Name not empty && !IsLiveBuildInProgress && no conflicts && Errors==0 && !empty && CanPreview
- ✅ `ConflictMessage` set by `CheckConflicts()` with `_existingMacroNames`

---

## 2. Context Menu & Step Card Actions

### Full Context Menu (ListBoxItem, lines 1381-1686)

| Menu Item | Command | Visibility | Verdict |
|-----------|---------|------------|---------|
| Preview Step | `UnifiedPreviewCommand` | Context-sensitive per type (ImageSearch, PixelSearch, UIElement, WindowAction, SystemSound, Popup, Notification, FileLauncher, UserInput, WaitForKey) | ✅ PASS |
| Rename | `EditStepNameCommand` | Always visible | ✅ PASS — Opens InputPromptWindow for rename, preserves LogicSource references |
| Duplicate Step | `DuplicateStepCommand` | Always visible | ✅ PASS — Clones step + associated files, handles VirtualSourceSteps |
| Disable/Enable Step | `ToggleDisableStepCommand` | Always visible | ✅ PASS — Cascade disable to children |
| Human Flow | `ToggleHumanizationStepCommand` | Only when `GlobalHumanizationEnabled=true` | ✅ PASS |
| Edit Coordinates | `EditCoordinatesCommand` | MouseClick, MouseTrace only | ✅ PASS |
| Edit End Coordinates | `EditEndCoordinatesCommand` | ImageSearch + Drag drops only | ✅ PASS |
| Move Up | `MoveStepUpCommand` | Always visible | ✅ PASS |
| Move Down | `MoveStepDownCommand` | Always visible | ✅ PASS |
| Move (into container) | Dynamic submenu | Always visible | ⚠️ Placeholder "Scanning..." populated at runtime via `ContextMenuOpening` |
| Group Selected (Ctrl+G) | `GroupSelectedCommand` | Collapsed by default, shown when >1 step selected | ✅ PASS |
| Ungroup (Ctrl+Shift+G) | `UngroupCommand` | GroupHeader, LoopSequence, LogicIf only | ✅ PASS |
| Delete Step | `RemoveStepCommand` | Always visible, red icon | ✅ PASS |

### Context Menu Conditions

- **Preview Step** has 12+ DataTriggers/DataTrigger conditions controlling visibility per step type — the most complex element in the menu
- Separator before Build Group is only visible for ImageSearch, PixelSearch, UIElement, WindowAction
- **All icons use `DynamicResource` colors** — no hardcoded hex values in context menu icons

### Step Card Layout (MacroStepCard.xaml, 949 lines)

| Element | Lines | Verdict |
|---------|-------|---------|
| DragHandle | 156 | ✅ Near-transparent `#01FFFFFF` overlay, 26×16 px, cursor=Hand, tooltip "Drag to reorder block" |
| ExpandToggleLeft | 159 | ✅ Collapsed by default, bound to `IsExpanded` (collapsed because ShowAccentBars handles expand) |
| IconContainer | ~110 | ✅ 30×30 rounded icon + WarningDot |
| StepTitle | ~120 | ✅ TextBlock bound to `StepName` |
| Inline panels | ~130-300 | ✅ ~20 ContentControl lazy-loaded panels per step type |
| SummaryContainer | ~130 | ✅ DisplayValue with HotkeyConverter, per-type summaries |
| ConfigContainer | ~300+ | ✅ Collapsible parameter zone with 20+ step-specific panels |
| Delete button (OptionHandle) | 387-405 | ✅ Only right-side control, red on hover |
| ChildStepsContainer | 408-460 | ✅ YES branch, `TokenLogicYesBrush` border, nested ItemsControl |
| ChildStepsFalseContainer | 461-507 | ✅ NO branch, `TokenLogicNoBrush` border, nested ItemsControl |
| Depth indentation | 73 | ✅ `DepthToMarginConverter` on root Grid |

### Drag and Drop (MacroEditorView.xaml.cs)

- ⚠️ `_dragStartPoint`, `_isDragging`, `_canInitiateDrag` fields
- ✅ DropIndicatorLine (line 1301-1307) — blue border line for visual placement
- ✅ DragGhostBox (line 1310-1325) — floating ghost for Premiere Pro feel
- ✅ Multiple event handlers: `PreviewMouseLeftButtonDown`, `MouseMove`, `PreviewMouseLeftButtonUp`, `Drop`, `DragEnter`, `DragOver`, `DragLeave`
- ✅ `TimelineListBox.AllowDrop=True`

### Key Commands (Core.cs)

| Command | Lines | Verdict |
|---------|-------|---------|
| `AddNewStep()` | 244-311 | ✅ Creates step, finds insert position (after selected or parent), handles Mouse→Image/Pixel auto-target, pushes Undo state |
| `RemoveStep()` | 316-389 | ✅ Undo push, VirtualSourceSteps handling (shared modifier protection), file cleanup, dispatcher-invoked |
| `DuplicateStep()` | 453-523 | ✅ Clones with unique name (`GetUniqueDuplicateName`), duplicates files, handles VirtualSourceSteps |
| `MoveStep()` | 585-638 | ✅ VirtualSourceSteps-aware move, `ObservableCollection.Move()` for simple steps |
| `NestStep()` | 640-711 | ✅ Nests into preceding container (LogicIf/GroupHeader/LoopSequence), handles both branches |
| `NestStepInto()` | 804+ | ✅ Moves step into specific named container anywhere in the tree |
| `UnnestStep()` | 848+ | ✅ Reverse of Nest — extracts step from container |
| `GroupSelectedSteps()` | 1105+ | ✅ Groups multiple selected steps into a GroupHeader |
| `UngroupStep()` | 1165+ | ✅ Ungroups, preserves children |
| Undo/Redo | Commands.cs:107-161 | ✅ Flush pending, stop watching, replace collection, force refresh, restart watching |

---

## 3. Recording Engine

### Architecture

**File:** `MacroEditorViewModel.Recording.cs` (774 lines)

| Component | Verdict |
|-----------|---------|
| `ToggleRecordCommand` | ✅ Calls `StartRecording()` / `_recorder.StopRecording()` |
| `StartRecording()` (lines 114-699) | ✅ Orchestrator: stops engine, resolves insert position, pushes Undo, creates `MacroRecordingService` on background task |
| `MacroRecordingService` | ✅ Third-party recording service with background task |
| `Cleanup()` (lines 752-771) | ✅ Stops recording, closes widget on view unload |
| `DetectLoopPattern()` (lines 708-747) | ✅ Analyzes recorded steps for 3+ repeats, suggests Repeat blocks |
| `SwapRecordedSteps()` (lines 59-113) | ✅ Swaps between No-Trace and All-Trace modes |

### Recording Features

| Feature | Status | Details |
|---------|--------|---------|
| Mouse path tracking | ✅ | `MouseTrace` + `TraceFileId`, stored in `TraceData` directory |
| Keypress capture | ✅ | `Keyboard` type steps |
| Smart delay merging | ✅ | Accumulated during recording callback, flushed on stop |
| Dual-mode storage | ✅ | `_recordedNoTraceSteps` + `_recordedAllTraceSteps` |
| Recording view mode | ✅ | `RecordingViewMode` (0=No Trace, 2=Full Trace), `CycleRecordingViewModeCommand` |
| Floating widget | ✅ | `RecordingWidgetView` shown during recording |
| Loop detection | ✅ | `DetectLoopPattern()` analyzes for repeating patterns |
| Auto-pause between steps | ✅ | `AutoDelayEnabled` + `AutoDelayPreset` |
| Performance mode disable | ✅ | Shimmers disabled in PerformanceMode |
| Trace settings | ✅ | `RecordIdleMouseMovement`, `CaptureMousePosition`, `CaptureKeyboardInput`, `CaptureWindowSwitches` |

### Recording Settings (Properties.cs)

- `IsRecording`, `HasDualModeData`, `RecordingViewMode`, `CycleRecordingViewModeCommand`
- `RecordTraceOff`, `IsRecordMousePathEnabled`, `RecordAllMovement`
- `CaptureMousePosition`, `CaptureKeyboardInput`, `CaptureWindowSwitches`, etc.
- `FastScrollPreview`, `FastClickPreview`, `FastTypingPreview` (preview speed toggles)
- `GlobalHumanizationEnabled`, `DefaultHumanizationLevel`, `HumanizationDropdownIndex`
- `HoldDelayPreset`, `PlaybackSmoothMouse`, `PlaybackRawMouse`, `PlaybackHumanSpeed`
- `IsSmartMode`, `IsStepFilterActive`, `SelectedFilterType`

---

## 🎯 Issues Found

### 🔴 Issue 1: Hardcoded `#FFFFFF` in MacroStepCard.xaml (3 instances)

| Line | Code | Element |
|------|------|---------|
| 44 | `Foreground="#FFFFFF"` | `ParamTextBoxStyle` — parameter textbox foreground fixed to white |
| 156 | `Background="#01FFFFFF"` | `DragHandle` hit-test overlay (near-invisible — acceptable, but still hardcoded) |
| 184 | `Foreground="#FFFFFF"` | `ExpandToggle` icon hover foreground |
| 454 | `Fill="#0AFFFFFF"` | Dashed drop-zone placeholder fill in ChildStepsContainer |
| 507 | `Fill="#0AFFFFFF"` | Dashed drop-zone placeholder fill in ChildStepsFalseContainer |

**Impact:** Light Mode would show white text on white/light background for `ParamTextBoxStyle`.

### 🔴 Issue 2: Hardcoded `#FFFFFF` in MacroEditorView.xaml (1 instance)

| Line | Code | Element |
|------|------|---------|
| 468 | `Foreground="#FFFFFF"` | Add Step toggle checked state text |

**Impact:** White text on light background in Light Mode.

### 🔴 Issue 3: AI Live Build Rainbow Colors Hardcoded (lines 235-248)

| Color | Usage |
|-------|-------|
| `#A78BFA` | Purple keyframe in rainbow animation |
| `#5AC8FA` | Blue keyframe in rainbow animation |
| `#FF4D4D` | Red keyframe in rainbow animation |

**Impact:** These are animation keyframes, not static colors — acceptable for a decorative animation but won't adapt to Light Mode.

### ⚠️ Issue 4: No Preview Button on Step Card

The step card (MacroStepCard.xaml) has **no inline preview/play button**. Preview is only accessible via:
1. Right-click context menu → Preview Step
2. The header toolbar's Preview Macro button (full macro preview only)

**Impact:** Poor UX — users must right-click to preview an individual step. No one-click preview.

### ⚠️ Issue 5: "Move" Submenu Uses Runtime Population

The "Move" menu item (line 1646-1653 in XAML) has:
```xml
<MenuItem Header="Scanning..." IsEnabled="False"/>
```
This is populated dynamically by the `ContextMenuOpening` event handler calling `GetAllNestTargets()`.

**Impact:** If the handler fails or returns null, the user sees "Scanning..." permanently. Single point of failure with no error fallback.

### ✅ Issue 6: ContextMenuOpening Handler — EXISTING (confirmed false flag)

The handler `OnTimelineContextMenuOpening` exists at `MacroEditorView.Events.cs:850`. It populates the Move submenu via `PopulateMoveInsideMenu()`.

### ✅ Issue 7: Recording Uses Dispatcher.InvokeAsync Extensively

`StartRecording()` uses `Dispatcher.InvokeAsync` for all UI updates during recording. While correct, this creates a tight coupling between recording performance and UI thread responsiveness.

---

## Summary

| Area | Verdict |
|------|---------|
| Header toolbar layout | ✅ PASS — All buttons, toggles, and commands properly wired |
| Add Action popup | ✅ PASS — 18+ types, 3 categories, drag-to-add |
| Empty timeline state | ✅ PASS — Clean, informative, theme-aware |
| Preview button enablement | ✅ PASS — `CanPreview` validates steps recursively |
| Save button enablement | ✅ PASS — `IsSaveReady` checks 6 conditions |
| Context menu — all items | ✅ PASS — 13 items, type-sensitive visibility |
| Context menu — icons/colors | ✅ PASS — All DynamicResource tokens |
| Drag handle reordering | ✅ PASS — Near-transparent overlay, drag ghost, drop indicator |
| Container nesting | ✅ PASS — Depth indentation, expand/collapse, YES/NO branches |
| Undo/Redo manager | ✅ PASS — 50-level history, full push/restore/replace cycle |
| Recording engine | ✅ PASS — Dual-mode, loop detection, smart delays, trace settings |
| **Hardcoded `#FFFFFF` in MacroStepCard.xaml** | 🔴 **3 instances** |
| **Hardcoded `#FFFFFF` in MacroEditorView.xaml** | 🔴 **1 instance** |
| **AI build indicator hardcoded colors** | 🔴 **3 hex values** |
| **No preview button on step card** | ⚠️ **UX gap** |
| **Move submenu runtime population** | ⚠️ **No error fallback** |
| **OnTimelineContextMenuOpening handler** | ✅ **Confirmed existing** (Events.cs:850) |

---

## 4. Additional Deep Re-Scan Findings

### 4a. 🔴 CRITICAL: 5 Undefined StaticResources That Will Crash at Runtime

| Resource | Referenced At | Scope | Impact |
|----------|---------------|-------|--------|
| `PremiumComboBoxStyle` | MacroEditorView.xaml:1091,1188,1218,1243 and 34+ other locations (SettingsDashboardView.xaml, CustomActionCard.xaml, SettingsView.xaml, MacroEditorOverlays.xaml:146) | **Never defined anywhere** | `KeyNotFoundException` when any referencing view renders |
| `PanelBackgroundBrush` | MacroEditorView.xaml:423 | **Never defined anywhere** | Toolbar border becomes invisible (no exception — fallback to default) |
| `BorderSubtleBrush` | MacroEditorView.xaml:423 | **Never defined anywhere** | Same — falls back silently |
| `RecordPillButton` | MacroEditorView.xaml:435 | **Never defined anywhere** | Record button gets default WPF look instead of pill style |
| `DropdownMenuCheckBoxStyle` (in Overlays) | MacroEditorOverlays.xaml:90 | Defined inside MacroEditorView.xaml (68-110) — different UserControl scope | **Will crash** — cross-UserControl StaticResource resolution fails |

### 4b. 🔴 CRITICAL: Encoding Corruption — Mojibake in MacroEditorView.xaml:270

```
Text="Ã¢Å“Â¨"   ← UTF-8 bytes E2 9C A8 (✨ U+2728) mis-interpreted as Windows-1252
```

Expected `✨` (sparkle). Renders as garbage text. Use `&#x2728;` or MDL2 icon.

### 4c. 🔴 CRITICAL: SolidColorBrush Color Type Mismatch Risk (MacroEditorView.xaml:238,275)

```xml
<SolidColorBrush Color="{DynamicResource TokenPurple300}" />
<SolidColorBrush Color="{DynamicResource TokenBlue500}" />
```

If `TokenPurple300`/`TokenBlue500` are defined as `SolidColorBrush` resources (not `Color`), the `SolidColorBrush(Color)` constructor throws an exception. The tokens in DarkTheme.xaml/LightTheme.xaml define **both** — e.g., `<Color x:Key="TokenPurple300">#A78BFA</Color>` AND `<SolidColorBrush x:Key="TokenPurple300Brush" Color="{StaticResource TokenPurple300}" />`. The key `TokenPurple300` (without "Brush" suffix) IS a Color, so this should work — **but only if the theme file is loaded first**. If loaded out of order, could crash.

**Same risk at** MacroEditorView.xaml:275,319 and MacroEditorOverlays.xaml:1312.

### 4d. 🔴 HIGH: IsSelected Binding Conflict (MacroEditorView.xaml:1379)

```xml
<Setter Property="IsSelected" Value="{Binding IsSelected, Mode=TwoWay}"/>
```

WPF's `ListBoxItem` already manages `IsSelected` internally. Overriding with an explicit binding **silently conflicts** — the model property and the ListBox's internal selection state can diverge, causing multi-select, drag, and context menu bugs.

### 4e. 🔴 HIGH: Smart View — `TryBuildShortcutBlock` Absorbs Arbitrary Steps (SmartView.cs:364-366)

Any non-keyboard/delay step type between a modifier-hold and modifier-release is absorbed into the shortcut block's `innerSteps`. This means mouse clicks, image searches, or other actions happening between key-down and key-up would be **incorrectly bundled** into the keyboard shortcut rather than emitted as independent steps.

### 4f. 🔴 HIGH: Smart View — `MapSnapshotToSteps` Mutates Shared State (SmartView.cs:554-558)

`MapSnapshotToSteps` mutates `OriginalStep.Depth` and `OriginalStep.VirtualSourceSteps` on the **original step objects**. If `BuildSmartSteps()` runs multiple times (e.g., during live editing), `VirtualSourceSteps` accumulates stale entries from previous runs.

### 4g. ⚠️ HIGH: Move Submenu Smart-Hide Only Checks `ChildSteps` (Events.cs:1108)

```csharp
if (target.Type != StepType.LogicIf && target.ChildSteps?.Contains(step) == true)
```

For Group/LoopSequence containers, this only checks `ChildSteps` (true branch), **not `ChildStepsFalse`**. A step already inside the false branch of a loop/group would still be offered as a move target.

### 4h. ⚠️ HIGH: Hardcoded RGB Brushes in Move Submenu Code-Behind (Events.cs:972-1020)

8 `SolidColorBrush` instances created in C# code with hardcoded RGB values:

| Variable | Line | RGB |
|----------|------|-----|
| `purpleBrush` | 974 | `0xA7, 0x8B, 0xFA` |
| Separator | 993 | `0x2D, 0x2F, 0x36` |
| Disabled item | 1004-1005 | `0x88, 0x8C, 0x96` |
| `greenBrush` | 1012 | `0x34, 0xC7, 0x59` |
| `redBrush` | 1014 | `0xFF, 0x3B, 0x30` |
| `yellowBrush` | 1016 | `0xF5, 0xC5, 0x18` |
| `blueBrush` | 1018 | `0x5A, 0xC8, 0xFA` |
| `loopPurpleBrush` | 1020 | `0xAF, 0x52, 0xDE` |

**Impact:** These are NOT theme-aware. In Light Mode, all these brushes stay the same dark-theme colors.

### 4i. 🔴 HIGH: RecordingWidgetView Window Flash at (0,0) (RecordingWidgetView.xaml)

- No `WindowStartupLocation` set — defaults to `Manual`
- Position corrected in `Loaded` event via `_savedLeft`/`_savedTop`
- **Brief visual flash at top-left corner** before Loaded fires
- `_savedLeft`/`_savedTop` are **static** — persist across app sessions

### 4j. ⚠️ MEDIUM: RecordingWidgetView BorderBrush Uses Wrong Token (RecordingWidgetView.xaml:10)

```xml
BorderBrush="{DynamicResource TokenCardBgBrush}"
```

Using a **background brush** as a border brush. Likely intended to be `TokenBorderDefaultBrush`.

### 4k. ⚠️ MEDIUM: `ParamTextBoxStyle` Defined in 3 Places — Inconsistent Resolution

| Location | Lines | Scope | Type |
|----------|-------|-------|------|
| App.xaml | 1101 | Global | DynamicResource |
| MacroStepTemplates.xaml | 19 | Merged dictionary | StaticResource/DynamicResource |
| MacroStepCard.xaml | 42 | ControlTemplate local | StaticResource |

Some controls reference `{StaticResource ParamTextBoxStyle}` while others use `{DynamicResource ParamTextBoxStyle}`. This means some step parameter textboxes resolve to the card-local style and others to the app-level style — potentially producing different visual results.

### 4l. ⚠️ MEDIUM: Skeleton Shimmer — 9 Perpetual Animations Running Simultaneously

`ProcShimmer1/2/3`, `SwitchShimmer1/2/3`, `ShimmerBrush1/2/3`, `ShimmerBrush0` — each animates 3 gradient stops at 1.4s intervals with `RepeatBehavior="Forever"`. **Unnecessary GPU/CPU overhead** while editor is open.

### 4m. ⚠️ MEDIUM: Multiple Empty Catch Blocks (DragDrop.cs)

| Line | Location | Risk |
|------|----------|------|
| 201 | Lasso canvas transform | Silently aborts lasso this frame |
| 224 | Per-item transform in lasso | Silently skips item |
| 433 | Gap-snap during DragOver | Silently falls through |
| 916 | Gap-snap during Drop | **Completely empty** — no logging |

### 4n. ⚠️ MEDIUM: Optimization — Hardcoded Step Windows

| Issue | File:Line | Detail |
|-------|-----------|--------|
| WindowAction reorder window | Optimization.cs:75 | Scans backwards `Math.Max(0, i-3)` — if click is 4+ steps before Activate, reorder fails |
| Orphaned Release stripping window | Optimization.cs:147 | Scans backwards `Math.Max(0, i-5)` — if Hold is outside 5-step window, Release is falsely stripped |
| `CleanupMacroSteps` is dead code | Optimization.cs:190-199 | Method body is empty after `ToList()` — iterates but never modifies |

### 4o. ⚠️ MEDIUM: Smart View Inconsistent Click Bundling Thresholds (SmartView.cs)

| Lines | Threshold | Purpose |
|-------|-----------|---------|
| 136-137 | **5px** | Initial same-spot detection for click bundling |
| 156-157 | **10px** | Same-spot detection for multi-click lookup |
| 213-214 | **150ms** | Scroll bundling delay threshold |
| 287-288 | **800ms** | Simple hold+release cleanup delay |
| 493 | **800ms** | Text bundling delay threshold |

**Inconsistency:** Initial click match uses 5px, multi-click lookup uses 10px. No documented reason for the difference.

### 4p. ⚠️ MEDIUM: Cheat Sheet Unclosable Vulnerability (MacroEditorCheatSheet.xaml.cs:17-25)

The backdrop click handler is intentionally empty ("Do not close on empty area click. Must use X or Escape."). The only close path is `ToggleCheatSheetCommand`. If the command binding is broken, **the cheat sheet is permanently open** and blocks all interaction.

### 4q. ⚠️ MEDIUM: UserControl Crashing If `OverlaysContainer` Is Not Founded In the Visual Tree

- Events.cs:273-282: `OverlaysContainer?.HideAllDialogs()` — safe via null-conditional
- But if the OverlaysContainer is genuinely missing, the `ForceExit` and `ConfirmSave` dialogs won't show — user is trapped in the editor

### 4r. ⚠️ LOW: Hardcoded Colors in MacroEditorView.xaml (Additional)

| Line | Value | Element |
|------|-------|---------|
| 44 | `#3B3D46` | DropdownMenuItem hover bg enter animation |
| 51 | `#101115` | DropdownMenuItem hover bg exit animation |
| 45 | `#FFFFFF` | DropdownMenuItem hover foreground |

### 4s. ⚠️ LOW: `async void` Save Handler Crash Risk (Events.cs:657)

`ConfirmSave_Click` is `async void` — if an exception occurs after an `await`, the process crashes. No try/catch wrapping.

### 4t. ⚠️ LOW: Toast Animation Overlap (MacroEditorOverlays.xaml:307-357)

Two separate Storyboard instances for enter/exit. If `IsToastVisible` toggles faster than animation duration (0.3-0.4s), exit and enter animations overlap — visual corruption.

### 4u. ⚠️ LOW: Popup Debounce Prevents Re-Open Within 500ms (Events.cs:14,64,73)

The `_popupLastClosedTime` tracking prevents popup re-open within 500ms. If a user clicks to open, immediately closes, and clicks again within 500ms — nothing happens. May feel unresponsive.
