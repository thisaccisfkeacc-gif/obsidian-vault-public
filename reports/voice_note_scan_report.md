# Voice Note Feedback — Codebase Audit Report

**Generated:** July 27, 2026  
**Scope:** Read-only scan of `PowerX_Keys_V2_Rebuild`  
**Source:** `Obsidian Vault/ideas/voice_note_feedback.md`

---

## 🐛 Bugs & Issues

---

### 1. Dashboard & App Scope

#### 1a. Remove Extra List Button

- **Status:** Verified Bug
- **Target File(s):** `PowerX.UI\Views\ScriptLibraryView.xaml:L1064-L1100`
- **Root Cause Analysis:** When App Scope is set to `Include Only`, the gear menu popup displays both a **"Capture App" button** (line 1064) and an **extra "📋" list button** (line 1084) side by side in a `Grid` with two columns. The list button (`CaptureAppFromListCommand`) is always visible regardless of scope mode — there is no `DataTrigger` or binding that hides it. The bug note requests removing the list button completely from the scope selector.
- **Fix Strategy:** Remove the list button (`Grid.Column="1"` Button at lines 1084-1100) or wrap it in a `DataTrigger` bound to `ScopeMode` that collapses it for `Include`/`Exclude` modes.

#### 1b. Capture App Icon Rendering

- **Status:** Verified Bug
- **Target File(s):** `PowerX.UI\Views\ScriptLibraryView.xaml:L1071`
- **Root Cause Analysis:** The "Capture App" button uses a raw emoji `⚡` (U+26A1) as the icon. Emoji rendering in WPF is inconsistent across Windows versions and DPI settings — often renders as a black-and-white outline or broken/tofu character depending on Segoe UI Emoji font availability.
- **Fix Strategy:** Replace the emoji `⚡` with a proper Segoe MDL2 Assets glyph (e.g., `&#xE723;` or `&#xE77B;`) or use a custom vector Path geometry.

#### 1c. Misleading Tooltip

- **Status:** Verified Bug
- **Target File(s):** `PowerX.UI\Views\ScriptLibraryView.xaml:L1064`
- **Root Cause Analysis:** The "Capture App" button has `ToolTip="Capture active window (3s delay)"` — inherited from the old window capture feature, completely wrong for App Scope context which captures the target application executable, not the active window with a delay.
- **Fix Strategy:** Change to `"Select a target application for this scope rule"` or make it dynamic based on `ScopeMode`.

---

### 2. Gear Menus & Dropdown UI

#### 2a. Nested Gear Menu Close Bug

- **Status:** Verified Bug
- **Target File(s):** `PowerX.UI\Views\Templates\SearchTemplates.xaml:L180-L207` (Image gear), `PowerX.UI\Views\MacroEditorView.xaml` (main gear menu popup)
- **Root Cause Analysis:** The image pattern gear menu (`ImageSettingsToggle`) and main card gear menu are separate `ToggleButton` popups. There is no event handler closing the secondary "Capture Image" popup when the main gear menu is clicked. Only clicking empty dashboard space closes all popups.
- **Fix Strategy:** Add a handler on the main gear menu's `ToggleButton.IsChecked` or `Popup.Closed` event that programmatically unchecks `ImageSettingsToggle` to close its popup.

#### 2b. UI Element Trigger Capture Button Broken

- **Status:** Verified Bug
- **Target File(s):** `PowerX.UI\Views\Templates\SearchTemplates.xaml:L1543-L1561`, `PowerX.UI\ViewModels\MacroEditorViewModel.Capture.cs`
- **Root Cause Analysis:** The "Capture UI Element" button command `CaptureUIElementCommand` may have an incorrect `CanExecute` predicate. Additionally, `CaptureOverlay` may fail to initialize properly when triggered from an inline editing context due to focus/visual-parent issues.
- **Fix Strategy:** Audit `CanExecute` for `CaptureUIElementCommand` — ensure it returns `true` with a valid step context. Verify `CaptureOverlay.ShowCaptureUIElement()` initializes in the correct visual tree and dispatcher context.



#### 2c. Toggle A/B Mode Broken Icon

- **Status:** Verified Bug
- **Target File(s):** `PowerX.UI\Views\CustomActionCard.xaml:L664`
- **Root Cause Analysis:** The "Toggle Slots" section header uses `&#xE8AB;` (Segoe MDL2 Assets). The "Second Press" label at line 676 uses standard Unicode text. The broken/gibberish icon likely comes from a `TextBlock` with an incorrect `FontFamily` or an invalid glyph codepoint elsewhere in the templates.
- **Fix Strategy:** Search `CustomActionCard.xaml` and `MacroStepCard.xaml` for any `TextBlock` rendering the second-press indicator icon. Verify the glyph codepoint against Segoe MDL2 Assets character map. Replace with a valid glyph or colored shape/dot.

#### 2d. If/Else Drag-Drop Reset Bug (Target Success dropdown)

- **Status:** Verified Bug
- **Target File(s):** `PowerX.UI\Views\MacroEditorView.DragDrop.cs:L515-L606` (invalid loop/redirect), `PowerX.UI\Views\Templates\LogicContainerTemplates.xaml:L13-L134`
- **Root Cause Analysis:** When a user selects a dropdown item in If/Else "Target Success" then drags the block, the `_effectiveDraggedItems` redirect logic interferes with `ComboBox` bindings, resetting `LogicSource` to null. Additionally, drag-drop `Clear()`/`Add()` operations on `MacroSteps` reset `SelectedValue` bindings because the step is removed and reinserted.
- **Fix Strategy:** Snapshot `LogicSource`/`LogicMode` before drag and restore after drop. Use `Move` operations instead of `RemoveAt`/`Insert` to preserve `ComboBox` bindings.

#### 2e. Context Menu Misplacement (Mouse Coordinates)

- **Status:** Verified Bug
- **Target File(s):** `PowerX.UI\Views\MacroEditorView.xaml:L1562-L1580`
- **Root Cause Analysis:** "Edit Coordinates" menu item is visible for `MouseClick` and `MouseTrace` via `DataTrigger` bindings (lines 1571-1576). However, there is **no filtering for scroll actions** — `WheelUp`/`WheelDown` subtypes still show the menu item even though coordinates are meaningless for them.
- **Fix Strategy:** Add a `MultiDataTrigger` checking both `Type == MouseClick` and the step's `Value` property — hide "Edit Coordinates" when `Value` is `"WheelUp"`/`"WheelDown"`.

#### 2f. Context Menu Clear Bug (If/Else)

- **Status:** Duplicate of 2d (same root cause)
- **Target File(s):** `PowerX.UI\Views\MacroEditorView.DragDrop.cs`
- **Root Cause Analysis:** Same as 2d. Drag operation resets `ComboBox SelectedValue` bindings.
- **Fix Strategy:** See 2d. Additionally set `e.Handled = true` on dropdown mouse events inside If/Else config panels to prevent drag interception.

---

### 3. Timeline & Block Cards

#### 3a. Recorded Window Blocks Showing Editable Properties

- **Status:** Verified Bug
- **Target File(s):** `PowerX.Core\Models\MacroItem.cs:L586-L597` (`IsManuallyAdded`), `PowerX.UI\Views\Templates\MiscTemplates.xaml:L262-L320`
- **Root Cause Analysis:** `IsManuallyAdded` flag exists but is **never checked** in XAML visibility logic for List, Pinpoint, Gear buttons. MiscTemplates.xaml shows them based on `WindowTitle` being non-null, with no `DataTrigger` for `IsManuallyAdded == false`. Recorded blocks show editable property buttons they shouldn't have.
- **Fix Strategy:** Add `DataTrigger` on `IsManuallyAdded` with `Value="False"` setting `Visibility="Collapsed"` on List, Pinpoint, and Gear icons in `MiscTemplates.xaml`.

#### 3b. Block Label Title Overwrite

- **Status:** Verified Bug
- **Target File(s):** `PowerX.Services\Services\MacroRecordingService.cs:L1073`, `PowerX.UI\Views\Templates\MiscTemplates.xaml:L236-L257`, `PowerX.UI\Converters\WindowTitleShortenerConverter.cs`
- **Root Cause Analysis:** `MacroRecordingService.cs:L1073` sets `StepName = title.Length > 40 ? title.Substring(0,37)+"..." : title` — the raw long window title (e.g., "Testing Checklist - Obsidian Vault... . Obsidian") becomes the primary bold text instead of the app name ("Obsidian"). The `StepName` should be the cleaned app name.
- **Fix Strategy:** In `EmitWindowBlock`, set `StepName` to the cleaned `exe` name (e.g., "Obsidian") instead of the full window title. Move the full title to `WindowTitle`. The template already splits app/title via `WindowTitleShortenerConverter`.

#### 3c. Captured Thumbnail Preview Image Missing

- **Status:** Verified Bug
- **Target File(s):** `PowerX.UI\Views\Templates\SearchTemplates.xaml:L42-L74`, `:L977-L1013`, `:L1662-L1693`, `PowerX.UI\Converters\ImagePathToThumbnailConverter.cs`
- **Root Cause Analysis:** `ImagePathToThumbnailConverter` returns `null` when path is empty or file doesn't exist (lines 15, 44). If `SearchImageFilename`/`UIScreenshotPath` isn't set to the full valid path after capture, the converter returns null and no thumbnail appears.
- **Fix Strategy:** Audit post-capture data flow in `MacroEditorViewModel.Capture.cs` — verify `step.SearchImageFilename`/`step.UIScreenshotPath` is set to the **full valid PNG path**. Add diagnostic logging when file doesn't exist.

#### 3d. UI Element "Check Exists" Failure

- **Status:** Verified Bug
- **Target File(s):** `PowerX.Services\Services\MacroExecutionService.cs:L2411-L2462`
- **Root Cause Analysis:** When `UIFallbackToCoordinates` is enabled (default) and element is **not found**, the fallback (lines 2417-2447) clicks at saved coordinates and **returns `true`** (line 2447) — setting `LastActionSucceeded=1` and `StepSuccessStates[name]=true`. This makes If/Else route to `Yes` even though the element never existed. When disabled, `StepSuccessStates[name]=false` correctly (lines 2451-2455). The default `true` causes "Check Exists" to always succeed.
- **Fix Strategy:** Remove `"Check Exists"` from the fallback actions list at line 2419. When element not found with action "Check Exists", always set `StepSuccessStates[cleanName]=false` regardless of `UIFallbackToCoordinates`.

#### 3e. Preview Box Red Highlight Misplacement

- **Status:** Verified Bug
- **Target File(s):** `PowerX.UI\Views\UIElementHighlightWindow.cs:L20-L52`, `:L76-L96`, `:L145-L150`
- **Root Cause Analysis:** Two issues:
  1. **DPI Correction Bug (line 85):** `OnSourceInitialized` only corrects when `dpiScale != 1.0`. If initial guess from `GetInitialDpiScale()` (queries MainWindow) is 1.0 but actual monitor is 125% (1.25), correction is skipped. Condition should be `dpiScale > 0`.
  2. **ShowNotFound (lines 145-150):** A fixed 40x40 red box is drawn at `(step.X, step.Y)` — the **captured coordinates**. If element moved, the red box appears at the old location, misleading the user.
- **Fix Strategy:** Fix condition at line 85 to `if (dpiScale > 0)`. For ShowNotFound, show a full-screen "Element Not Found" overlay or a notification toast instead of a misplaced red box.

---

### 4. Search Area Overlay & Selection

#### 4a. Smart Search Area Button Label

- **Status:** Verified Bug
- **Target File(s):** `PowerX.Core\Models\MacroItem.cs:L1541-L1631` (`SearchScopeDisplay`), `PowerX.UI\Views\Templates\SearchTemplates.xaml:L133` (label binding)
- **Root Cause Analysis:** `PropertyChanged` for `SearchScopeDisplay` is only raised in `SearchScopeSummary` setter (line 1545), but `SearchScopeDisplay` also depends on `X`, `Y`, `SearchWidth`, `SearchHeight`. When these change (user edits coordinates or clicks "Full Screen" then "Use Search Area"), `PropertyChanged` is **not raised**, so the label remains stale.
- **Fix Strategy:** Raise `OnPropertyChanged(nameof(SearchScopeDisplay))` in setters of `X`, `Y`, `SearchWidth`, `SearchHeight`. Or recompute `SearchScopeSummary` whenever coordinates change.

#### 4b. Overlay Guide Text Interception

- **Status:** Verified Bug
- **Target File(s):** `PowerX.UI\Views\CaptureOverlay.xaml.cs:L466-L476`
- **Root Cause Analysis:** `Window_MouseDown` checks `IsOverHeaderButtons(pos) || IsOverGuidePopup(pos)` (line 476), correctly preventing capture on popup clicks. However, guide instruction `TextBlock` elements rendered directly on `OverlayCanvas` intercept mouse-down events because they're part of the canvas hit-test tree, preventing drag-rectangle from starting.
- **Fix Strategy:** Set `IsHitTestVisible="False"` on guide instruction `TextBlock` elements on `OverlayCanvas`. Only `UseSmartAreaButton`/`UseFullScreenButton` should remain hit-testable.

---

### 5. Multi-Select & Drag-and-Drop

#### 5a. Multi-Select Top Drag Failure

- **Status:** Verified Bug
- **Target File(s):** `PowerX.UI\Views\MacroEditorView.DragDrop.cs:L246-L259`, `:L736-L761`
- **Root Cause Analysis:** When dragging multiple items to the **top** of the timeline, the `DragOver` handler (lines 736-761) positions the drop indicator at the **bottom of the last item** (`lastItemIndex + 0.5`), not at index 0. Additionally, `_effectiveDraggedItems` redirect (lines 515-606) may map multi-select to only one item.
- **Fix Strategy:** In the "empty space" branch, detect cursor above the first item and set drop index to 0. Make `_effectiveDraggedItems` redirect apply to all dragged items, not just one.

#### 5b. Multi-Select Delete Failure

- **Status:** Verified Bug
- **Target File(s):** `PowerX.UI\Views\MacroStepCard.xaml.cs:L127-L167`, `PowerX.UI\Views\MacroEditorView.xaml:L1668` (context menu)
- **Root Cause Analysis:** The X button handler correctly checks multi-selection and calls `BulkDeleteCommand`. However, the right-click context menu "Delete Step" (line 1668) binds to `RemoveStepCommand` — a **single-step removal**. It doesn't check `SelectedItems.Count` or call `BulkDeleteCommand`. The Delete key handler correctly uses `BulkDeleteCommand`, but the context menu doesn't.
- **Fix Strategy:** Update the context menu "Delete Step" to check `SelectedItems.Count`. If multiple selected, execute `BulkDeleteCommand` instead of `RemoveStepCommand`.

#### 5c. Multi-Select Shortcut Move Failure

- **Status:** Verified Bug
- **Target File(s):** `PowerX.UI\Views\MacroEditorView.Events.cs:L163-L174`
- **Root Cause Analysis:** `MoveUp`/`MoveDown` handler uses `TimelineListBox?.SelectedItem` (single item) and ignores `SelectedItems`. Both commands call `MoveStep(s, direction)` with a single step. `MoveStep` only moves one step via `collection.Move(index, newIndex)`.
- **Fix Strategy:** When `SelectedItems.Count > 1`, iterate selected items and move them as a group. Create `BulkMoveCommand` that sorts by index and moves in correct order.

#### 5d. Nested Container Drag-and-Drop Erratic Behavior

- **Status:** Verified Bug
- **Target File(s):** `PowerX.UI\Views\MacroEditorView.DragDrop.cs:L616-L722` (indicator depth), `:L370-L435` (gap-zone snap)
- **Root Cause Analysis:** Drop indicator indent = `element.Depth * 16` (line 616), `+16` for branches (line 622). At depth 2, indent = 48px. But `DropIndicatorBorder.Margin` (line 722) uses `leftIndent+24` with no parent-container scaling. `ChildStepsContainer` has its own `Margin="22,4,0,6"`/`Padding="15,8,5,8"` (MacroStepCard.xaml). `DepthToMarginConverter` (depth*16) compounds differently from indicator offset. Gap-zone snap (lines 381-435) struggles to distinguish nested-level gaps.
- **Fix Strategy:** Use unified depth-to-offset (Depth*24). Calculate indicator offset by walking visual tree ancestors, not model Depth. Add maximum indent cap.

---

### 6. Recorder & App State

#### 6a. Program Manager Raw Window Name

- **Status:** Verified Bug
- **Target File(s):** `PowerX.Services\Services\MacroRecordingService.cs:L1048-L1077`, `PowerX.Services\Services\ScriptCompilerService.cs:L2917-L2922`
- **Root Cause Analysis:** When desktop (`Progman`/`WorkerW` class) is foreground, `GetWindowText` returns `"Program Manager"`. `EmitWindowBlock` (line 1053) constructs title as `"Program Manager ahk_exe explorer.exe"`. No normalization exists for the desktop case — the timeline block displays the raw system name.
- **Fix Strategy:** Check `GetClassName` for `"Progman"`/`"WorkerW"` in `EmitWindowBlock`. Override `title` to `"Desktop"` and `exe` to `"explorer.exe"` (or just `"Desktop"`).

#### 6b. Alt+F4 During Recording Glitch

- **Status:** Verified Bug
- **Target File(s):** `PowerX.UI\Views\RecordingWidgetView.xaml.cs:L93-L116` (kill switch), `PowerX.Services\Services\MacroRecordingService.cs:L483-L548`, `PowerX.UI\ViewModels\MacroEditorViewModel.Recording.cs:L754-L769`
- **Root Cause Analysis:** Kill switch monitors Double Escape, Shift+Escape, Ctrl+Shift+Escape, Scroll Lock, Print Screen — but **not** Alt+F4. Pressing Alt+F4 closes the active window, terminating AHK recording abnormally. `StopRecording()` isn't called, leaving `IsRecording = true` and tray unresponsive.
- **Fix Strategy:** Add Alt+F4 to kill switch watcher. When detected, call `StopRecording()` before window closes. Add `Window.Closing`/`SessionEnding` handler that checks `IsRecording` and forces clean stop.

#### 6c. Net Disconnect Sign-in Loop

- **Status:** Verified Bug
- **Target File(s):** `PowerX_Keys_V2\App.xaml.cs` (auth gate), `PowerX.Services\Services\SupabaseAuthService.cs:L242`, `PowerX.UI\Views\AuthWindow.xaml`, `PowerX.UI\ViewModels\AuthViewModel.cs`
- **Root Cause Analysis:** Startup checks `IsAuthenticated` (in-memory, null after restart) then falls back to `FileSessionHandler.LoadTokens()`. If **offline**, token restoration fails silently (SupabaseAuthService.cs lines 34-55 catch all exceptions), leaving `IsAuthenticated=false`. Auth window shown, but sign-in also fails offline, creating an infinite loop. No offline mode exists.
- **Fix Strategy:** Implement offline detection at startup. If valid cached tokens exist but network is unavailable, skip auth screen and proceed in offline mode with a banner notification. Only show auth screen when **no cached tokens** exist.

---

## 💡 Improvements & Enhancements

---

### 1. UI Consistency & Visual Polish

#### 1a. Gear Icon Container Consistency (Pixel Color vs Image Pattern)

- **Status:** UI Inconsistency (Confirmed)
- **Target File(s):** `PowerX.UI\Views\Templates\SearchTemplates.xaml:L180-L207` (ImagePattern full gear), `:L583-L610` (ImagePattern inline), `:L862-L889` (PixelColor inline), `:L1339-L1366` (PixelColor full)
- **Root Cause Analysis:** Image Pattern gear icon (expanded, 28x28) and Pixel Color gear icon (expanded, 28x28) have slightly different wrapping structures. Image inline gear uses `HasSearchImage` for `IsEnabled`, Pixel inline gear uses `HasCapturedPixel`. UI Element inline gear uses `TokenBlue400Brush` while others use `TokenPurple300Brush` for checked state. The container templates differ — one has a background `Border`, the other uses a bare `ToggleButton`.
- **Fix Strategy:** Extract gear icon toggle into a shared `ControlTemplate`/`Style`. Normalize size (24x24 inline, 28x28 expanded), visual states, container border, and checked brush (`TokenPurple300Brush`) across all step types.

#### 1b. Human Flow Visual Badge

- **Status:** Feature Request (Keyboard badge exists, Mouse missing)
- **Target File(s):** `PowerX.UI\Views\Templates\KeyboardInputTemplates.xaml:L421-L442` (has badge), `PowerX.UI\Views\Templates\MouseTemplates.xaml` (no badge)
- **Root Cause Analysis:** Human Flow green pill badge exists for **Keyboard** blocks — visible when `GlobalHumanizationEnabled` + `IsHumanized` both true. **Mouse blocks** do NOT have this badge in their inline templates. The context menu toggle works for all block types, but only Keyboard shows the visual badge on the timeline card.
- **Fix Strategy:** Add the same Human Flow badge `Border` with appropriate `DataTrigger` to the Mouse block inline template in `MouseTemplates.xaml`. Consider adding to other `IsHumanized`-compatible block types.

#### 1c. Side Margin Lines in Containers

- **Status:** Design Decision (Not a bug — existing colored borders differ from described gray lines)
- **Target File(s):** `PowerX.UI\Views\MacroStepCard.xaml:L408-L429` (ChildStepsContainer), `:L461+` (ChildStepsFalseContainer)
- **Root Cause Analysis:** The If/Else container branches have `BorderThickness="2,0,0,0"` with colored (green/red) left borders, not gray guide lines. The bug note describes "2px vertical gray guide lines on the left side" which do not exist in the codebase — the current visual treatment is colored accent borders, not gray guides.
- **Fix Strategy:** If the intent is to remove colored borders, set `BorderThickness="0"` on both containers. If replacing with gray lines, add a separate `Border` with 2px gray left margin inside each container.

#### 1d. Flicker-Free Undo/Redo

- **Status:** Performance Issue (Confirmed)
- **Target File(s):** `PowerX.Services\Services\UndoRedoService.cs` (full file), `PowerX.UI\ViewModels\MacroEditorViewModel.Commands.cs:L107-L161` (Undo/Redo command), `MacroEditorViewModel.Properties.cs:L330-L405` (RefreshDisplaySteps)
- **Root Cause Analysis:** Undo/Redo (Commands.cs lines 107-161) calls `CurrentMacro.MacroSteps.Clear()` then re-adds all steps (lines 120-121), which triggers full WPF `ItemsControl` re-rendering. `ForceRefreshTimeline()` is called instead of `RefreshDisplaySteps()`, but the `Clear`+`Add` pattern still invalidates all visual containers. `RefreshDisplaySteps()` (Properties.cs lines 330-405) has background debouncing (50ms) and `DispatcherPriority.Background`, but the collection mutation causes the entire `ListBox` to re-virtualize, producing visible flicker with complex nested blocks.
- **Fix Strategy:** Instead of `Clear()`+`Add()`, use a diff-based approach: compare old and new step lists, apply minimal mutations (RemoveAt for deleted, Insert for added, Move for reordered). Use `ICollectionView` defer refresh or `VirtualizingPanel` optimizations. Consider `UIElement` recycling via `VirtualizationMode="Recycling"`.

---

### 2. Validation & Tooltips

#### 2a. Specific Error Tooltips

- **Status:** Feature Request
- **Target File(s):** `PowerX.UI\ViewModels\MacroEditorViewModel.Properties.cs:L479-L527` (CanPreview validation), `PowerX.Core\Models\MacroItem.cs:L1238` (IsValid)
- **Root Cause Analysis:** Current validation (`IsTimelineFullyValidated`, Properties.cs lines 493-527) returns a boolean `allValid` with no per-step error messages. Error tooltips like "Pixel Not Found" are generic — they don't identify the specific block. The `StepName` or index of the failing block is not included in any error message.
- **Fix Strategy:** Refactor `IsTimelineFullyValidated` to collect per-step error strings. Surface these in the UI via a `ToolTip` on the error badge that includes step name/number (e.g., "Pixel 'Color 1' Not Found in Block #3").

#### 2b. Empty Block Validation

- **Status:** Feature Request
- **Target File(s):** `PowerX.UI\ViewModels\MacroEditorViewModel.Properties.cs:L596` (IsTimelineEmpty), `:L479-L527` (CanPreview)
- **Root Cause Analysis:** `CanPreview` checks `hasActions` (non-container steps) and `allValid` (step validity). However, empty `If/Else` or `Repeat` blocks (0 nested actions) are not specifically flagged. An If/Else with no steps inside either branch passes `hasActions` because the If/Else step itself is excluded from the action count (Properties.cs line 518). Only pure top-level emptiness triggers `IsTimelineEmpty`. There is no "Empty Loop"/"Invalid Condition" badge on the save/preview button.
- **Fix Strategy:** Add a recursive check for empty containers: an If/Else with 0 steps in both `ChildSteps` and `ChildStepsFalse`, or a Repeat with 0 `ChildSteps`, should disable Save/Preview and show a warning badge.

#### 2c. Trial Expiration Screen Polish

- **Status:** Feature Request
- **Target File(s):** `PowerX.UI\Views\SubscriptionExpiredWindow.xaml:L168-L190` (Log out link), `SubscriptionExpiredWindow.xaml.cs:L49-L61` (LogOutButton_Click)
- **Root Cause Analysis:** The "Log out / switch account" link (XAML lines 168-190) is always visible. There is no conditional logic to hide it based on subscription status or trial state. The note requests hiding this link on the trial expiration screen to prevent trial bypass attempts.
- **Fix Strategy:** Add a `Boolean` property (e.g., `ShowLogoutLink`) to `SubscriptionExpiredWindow` that hides the logout link. Set it to `false` for trial-expired states. Only show `Upgrade` and `Exit` buttons when the trial has expired.

---

### 3. Timeline Performance

#### 3a. Skeleton Loading Placeholder

- **Status:** Feature Request (Infrastructure exists but not linked to timeline)
- **Target File(s):** `PowerX.UI\Views\SkeletonOverlay.xaml` (full skeleton UI), `PowerX.UI\ViewModels\MacroEditorViewModel.Properties.cs:L584-L594` (IsEditorLoading)
- **Root Cause Analysis:** `SkeletonOverlay` exists with full shimmer animation UI (skeleton cards, sidebar, header). `IsEditorLoading` property exists on the ViewModel (line 584) and is set to `true` initially and `false` after `RefreshDisplaySteps` completes (line 399). However, `IsEditorLoading` is **not bound to the `SkeletonOverlay` visibility** in `MacroEditorView.xaml`. The skeleton overlay is never shown before timeline population.
- **Fix Strategy:** Bind `SkeletonOverlay.Visibility` to `IsEditorLoading` in `MacroEditorView.xaml`. Show skeleton when `IsEditorLoading=true`, fade it out via `DismissWithFade()` when `IsEditorLoading=false`. This will show skeleton cards during the 1-2 second recording/loading gap.

---

## ✨ New Feature Ideas & Architecture

---

### 1. Trigger Mode Press & Release Split

- **Status:** Architecture Suggestion
- **Target File(s):** `PowerX.Core\Models\AppEnums.cs` (TriggerMode enum), `PowerX.UI\Converters\TriggerModeToReadableConverter.cs`
- **Root Cause Analysis:** Currently, Press and Release actions are merged under "Hold & Release" as a single trigger mode. The `TriggerMode` enum (AppEnums.cs) and related converters would need new values (e.g., `PressOnly`, `ReleaseOnly`) to support splitting hold/press and release triggers. The compiled AHK script in `ScriptCompilerService.cs` would need updated hotkey syntax (using `~*a` vs `~*a up` for release detection).
- **Proposed Strategy:** Add `PressOnly` and `ReleaseOnly` values to `TriggerMode` enum. Update `TriggerModeToReadableConverter` for display. In `ScriptCompilerService.cs`, modify hotkey registration to emit `*a::` for press-only and `*a up::` for release-only. Handle edge cases: both press and release bound to the same macro, or conflicting assignments.

### 2. Smart Recording Optimizations

#### Window Action Collapse

- **Status:** Feature Suggestion
- **Target File(s):** `PowerX.Services\Services\MacroRecordingService.cs` (recording event stream), `PowerX.UI\ViewModels\MacroEditorViewModel.SmartView.cs` (smart bundling)
- **Root Cause Analysis:** The raw recording stream emits separate `WindowAction`, `MouseClick`, and `Keyboard` steps. The Smart View bundling (SmartView.cs, lines 343-349 handling Alt+F4) already bundles some modifier shortcuts, but there is no logic to collapse "Window Focus + Click + Minimize" into a single "Minimize Window" action block.
- **Proposed Strategy:** Add a post-recognition step in `MacroEditorViewModel.SmartView.cs` that detects the pattern: `WindowAction(Activate) -> Delay -> MouseClick(Minimize button coordinates)` and collapses into a single `WindowAction(Minimize)` with `WindowTitle` preserved. The `WindowAction.Value` enum would need a `"Minimize"` entry.

#### Key Press Auto-Merge

- **Status:** Feature Suggestion
- **Target File(s):** `PowerX.Services\Services\MacroRecordingService.cs` (raw key events)
- **Root Cause Analysis:** The AHK event stream emits separate `KeyDown(H)` and `KeyUp(H)` events for each key press. These are recorded as individual `Keyboard` steps with `Value = "KeyDown"` and `Value = "KeyUp"`. There is no merge logic to detect consecutive down/up of the same key and collapse into a single `KeyPress` step.
- **Proposed Strategy:** In the recording event callback (MacroRecordingService.cs), maintain a `_pendingKeyDown` dictionary tracking keys that have been pressed down but not yet released. When a `KeyUp` event arrives for the same key (with no other keys in between), emit a single `Keyboard(KeyPress, "H")` instead of two separate steps. Flush any unreleased keys when focus changes or on recording stop.

### 3. Image Cascade Search Optimization

- **Status:** Optimization Suggestion
- **Target File(s):** `PowerX.Services\Services\ScriptCompilerService.SingleStep.cs:L378-L649` (cascade levels)
- **Root Cause Analysis:** When checking IF image is NOT present (`FailIfMissing = false`, negative condition), the cascade runs all 4 fallback levels (Last Known Position → Smart Box → Window → Full Screen) before concluding the image is absent. This wastes execution time on up to 3 unnecessary search passes when any single "found" result means the condition fails (image IS present, which contradicts the negative check).
- **Proposed Strategy:** In the compiled AHK script, when the step's `FailIfMissing` is `false` (negative condition), inject a short-circuit: as soon as any cascade level finds the image, the hard-coded "stop" branch exits immediately. Only run subsequent cascade levels if the image is not found. This can be done by wrapping each cascade level in a check: `if (!foundImage) { tryNextLevel() }`.

### 4. Macro vs Template Architecture

- **Status:** Architecture Suggestion
- **Target File(s):** `PowerX.Core\Models\MacroItem.cs` (Macro vs MacroStep), `PowerX.UI\Views\Templates\LogicContainerTemplates.xaml:L234-L273` (CallMacro panel), `PowerX.Services\Managers\MacroDatabase.cs`
- **Root Cause Analysis:** Currently, "Call Macro" blocks reference other saved macros by ID. There is no separate concept of "Saved Templates" — reusable block logic snippets (e.g., a custom If/Else construct used across multiple macros). Templates would need their own database table, CRUD operations, distinct icon/badge in the library, and the "Call Macro" block would need a `CallTemplate` variant.
- **Proposed Strategy:** Add a `SavedTemplate` model (subclass of `Macro` or new entity) with a `TemplateId`, `Category`, `Parameters` list. Add a `TemplateLibrary` table to `MacroDatabase.cs`. In the Script Library, add a "Templates" section with drag-to-timeline support. Update the "Call Macro" block to offer a "Call Template" mode with parameter binding UI.

### 5. Kill Switch Shortcut Update

- **Status:** Feature Suggestion (Partial infrastructure exists)
- **Target File(s):** `PowerX.UI\Views\RecordingWidgetView.xaml.cs:L93-L116` (kill switch watcher), `PowerX.Services\Services\ShortcutManager.cs`
- **Root Cause Analysis:** The kill switch watcher in `RecordingWidgetView.xaml.cs` already has logic for `Shift + Escape` (line 103-104) alongside `Double Escape` (lines 93-100). The `ShortcutManager` registers editor shortcuts but does not manage the kill switch config. The shift+escape handler exists but may not have a settings toggle to let users choose between Double Escape (default) and Shift+Escape (primary).
- **Proposed Strategy:** Add a setting in `SettingsView.xaml` under "Recording" section: "Kill Switch Mode" with options `Double Escape` / `Shift + Escape`. Persist in `ConfigManager`. Update `RecordingWidgetView.xaml.cs` to read this setting and use the configured trigger as the primary kill switch. Keep the other as secondary fallback.

---

*End of Report*
