# Verification Prompt — PowerX Keys Bug Fixes

## Instructions
Scan the codebase at `C:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild` and verify each fix below. For each item, report:
- **PASS** = fix is correctly implemented, no regressions
- **FAIL** = fix is missing, incorrect, or introduces a regression
- **WARN** = fix is present but has concerns (explain)

Search for each change in the specified files and report whether the expected code is present. Also check for any obvious issues (missing usings, type mismatches, syntax errors, broken bindings, etc.).

---

## Fix 1a: Remove extra list button in App Scope gear menu
**File:** `PowerX.UI\Views\ScriptLibraryView.xaml`  
**Expected:** In the App Scope gear menu popup, there should be NO extra `ListBox` or list-related button alongside the `Capture App` button. Look for the gear menu DataTemplate and verify only the `Capture App` button exists (no sibling list button).

## Fix 1b: Replace emoji ⚡ with Segoe MDL2 glyph
**File:** `PowerX.UI\Views\ScriptLibraryView.xaml`  
**Expected:** The `Capture App` button's icon should use Segoe MDL2 glyph `&#xE77B;` instead of emoji ⚡. Search for the button template and check its icon TextBlock.

## Fix 1c: Fix misleading tooltip
**File:** `PowerX.UI\Views\ScriptLibraryView.xaml`  
**Expected:** The "Capture App" button's ToolTip should say `"Select a target application for this scope rule"` (not the old `"Capture Active Window 3 seconds delay"`).

## Fix 2a: Close sibling gear popups
**File:** `PowerX.UI\Views\Templates\TemplateHandlers.cs`  
**Expected:** In the `SettingsPopup_Opened` method, there should be a visual-tree walk that finds and closes other open gear popups when one opens. Look for logic that iterates `Popup` siblings and sets `IsOpen = false`.

## Fix 2c: Fix broken em dash and glyph rendering
**File:** `PowerX.UI\Views\CustomActionCard.xaml`  
**Expected:** Any em dash characters (—) should be replaced with proper rendering. Search for any text that uses Segoe MDL2 Assets font family for the icon next to "Second Press".

## Fix 2d: Fix If/Else drag-drop dropdown reset
**Files:** `PowerX.UI\Views\MacroStepCard.xaml`  
**Expected:** Both YES branch `ItemsControl` (x:Name="NestedItemsControl") and NO branch `ItemsControl` (x:Name="NestedItemsFalseControl") should have `VirtualizingPanel.IsVirtualizing` and `VirtualizingPanel.VirtualizationMode` REMOVED. Their `ItemsPanelTemplate` should use `StackPanel` instead of `VirtualizingStackPanel`.  
**Also verify:** These ItemsControls are in the YES/NO branches inside the `LogicIf` container template and they host `ChildSteps` / `ChildStepsFalse` respectively.

## Fix 2e: Collapse Edit Coordinates for WheelUp/WheelDown
**File:** `PowerX.UI\Views\MacroEditorView.xaml`  
**Expected:** There should be a `DataTrigger` that collapses the "Edit Coordinates" menu item when step type is `WheelUp` or `WheelDown`. Search for `ContextMenu` or `MenuItem` with `EditCoordinates` and check if there's a DataTrigger on step type.

## Fix 3a: Collapse List/Pinpoint/Gear when IsManuallyAdded=False
**File:** `PowerX.UI\Views\Templates\MiscTemplates.xaml`  
**Expected:** The List button, Pinpoint button, and Gear button should have a `DataTrigger` that sets `Visibility="Collapsed"` when `IsManuallyAdded` is `False`.

## Fix 3b: Use cleaned exe name for StepName
**File:** `PowerX.Services\Services\MacroRecordingService.cs`  
**Expected:** Where `StepName` is assigned for window focus/capture steps, it should use the cleaned executable name (e.g., extracting just "Obsidian" from the full path) instead of the raw window title. Look for `StepName =` assignment and verify it uses `Process.GetProcessById` or similar to get the process name.

## Fix 3c: Add screenshot capture to UIElementCaptureService
**Files:** 
- `PowerX.Services\Services\UIElementCaptureService.cs`
- `PowerX.UI\ViewModels\MacroEditorViewModel.Capture.cs`  
**Expected:** In `UIElementCaptureService.BuildResult()`, there should be code that captures a screenshot (using `Bitmap` or similar) and saves it to a file, setting `ScreenshotPath` on the result. In `CaptureUIElementAsync`, after capture, `step.UIScreenshotPath` should be set to `result.ScreenshotPath`.

## Fix 3d: Fix Check Exists false negatives
**File:** `PowerX.Services\Services\MacroExecutionService.cs`  
**Expected:** The "Check Exists" evaluation path should have:
1. A relaxed UIA retry mechanism using `AutomationId` + `ControlType` matching
2. A point-based UIA fallback when `AutomationId` is null
3. `_stepSuccessStates[cleanName] = true` set on the fallback-success path (search for `_stepSuccessStates` to find the correct location)

## Fix 3e: Fix DPI correction condition
**File:** `PowerX.Services\Services\UIElementHighlightWindow.cs`  
**Expected:** The DPI correction condition should be `dpiScale > 0` (not `dpiScale != 1.0`). Search for where `dpiScale` is used in a conditional.

## Fix 4a: Fix search area label not updating
**File:** `PowerX.Core\Models\MacroItem.cs`  
**Expected:** In the setters for `X`, `Y`, `SearchWidth`, and `SearchHeight`, there should be `OnPropertyChanged(nameof(SearchScopeDisplay))` calls so the label updates when coordinates change.

## Fix 4b: Set IsHitTestVisible=False on overlay guide text
**File:** `PowerX.UI\Views\CaptureOverlay.xaml`  
**Expected:** The elements `GuideTitleIcon`, `GuideTitle`, `GuideSubDesc`, and `CollapsedIcon` should have `IsHitTestVisible="False"` set.

## Fix 5a: Multi-select top drag auto-expand group
**Files:**
- `PowerX.UI\Views\MacroEditorView.xaml.cs`
- `PowerX.UI\Views\MacroEditorView.DragDrop.cs`  
**Expected:** In `MacroEditorView.xaml.cs`, there should be a `DispatcherTimer` field `_expandOnDragTimer`. In `DragDrop.cs`, the `TimelineListBox_DragOver` method should have logic to start this timer when hovering over a collapsed `GroupHeader`/`LoopSequence`/`LogicIf` container, and expand it on timeout.

## Fix 5b: Context menu bulk delete
**Files:**
- `PowerX.UI\Views\MacroEditorView.xaml`  
- `PowerX.UI\Views\MacroEditorView.Events.cs`  
**Expected:** In the XAML, the Delete menu item should have `Tag="DeleteStepMenuItem"`. In `Events.cs`, `OnTimelineContextMenuOpening` should check selection count and switch between `RemoveStepCommand` (single) and `BulkDeleteCommand` (multi-select).

## Fix 5c: Multi-select shortcut move
**File:** `PowerX.UI\Views\MacroEditorView.Events.cs`  
**Expected:** The `MoveUp`/`MoveDown` keyboard handler should iterate all `SelectedItems` in correct order — top-to-bottom for up, bottom-to-top for down (to maintain relative order).

## Fix 5d: Nested container drag indicator depth offset
**File:** `PowerX.UI\Views\MacroEditorView.DragDrop.cs`  
**Expected:** Around line 637, the `leftIndent` calculation should NOT add extra `+16` when `element != null`. When `element == null` inside a container, there should be code that walks up from `currHit` to find the parent LogicIf step and uses `(ms.Depth + 1) * 16`.

## Fix 6a: StepName collision with unique IDs
**File:** `PowerX.Services\Services\MacroRecordingService.cs`  
**Expected:** When generating variable names for steps (especially with duplicate step names), there should be logic to append unique IDs or counters to prevent collisions. Look for where step variable names are generated.

## Fix 6b: Alt+F4 kill switch for recording widget
**Files:**
- `PowerX.UI\Views\RecordingWidgetView.xaml`
- `PowerX.UI\Views\RecordingWidgetView.xaml.cs`  
**Expected:** In XAML, there should be `PreviewKeyDown="Window_PreviewKeyDown"` on the Window. In code-behind, there should be a handler that detects Alt+F4 and calls `StopRecording()` or similar cleanup before closing.

## Fix 6c: Offline sign-in loop
**Files:**
- `PowerX.Services\Services\SupabaseAuthService.cs`
- `PowerX_Keys_V2\App.xaml.cs`  
**Expected in SupabaseAuthService.cs:**
1. `HasSavedTokens` and `IsOfflineMode` boolean properties
2. In `InitializeAsync()`, network errors (`HttpRequestException`, `TaskCanceledException`) should NOT clear tokens — only auth errors should
3. A subscription cache mechanism with `SaveCachedSubscription` / `LoadCachedSubscription` methods using `sub_cache.json` with 7-day TTL  
**Expected in App.xaml.cs:**
1. Auth check should skip auth window when `HasSavedTokens` is true
2. Subscription check should use cached subscription (with 3-day grace fallback) when offline

## Build Check
Finally, if possible, run `dotnet build` on the project and report any compilation errors:
```
cd "PowerX Keys\PowerX_Keys_V2_Rebuild"
dotnet build PowerX_Keys_V2\PowerX_Keys_V2.csproj
```

---

## Report Format
Return a structured report like this for each item:

```
### Fix Xx: Name
- File(s): path
- Status: PASS / FAIL / WARN
- Evidence: (what you found in the code)
- Issues: (any problems found)
```

At the end, give an overall summary: how many PASS/FAIL/WARN out of total.
