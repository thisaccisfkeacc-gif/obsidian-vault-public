[]()# 🎧 Voice Note Feedback & Task Breakdown

**Source File:** `New recording 17.m4a`  
**Location:** `C:\Users\Maaz\Downloads\Audios\New recording 17.m4a`  
**Duration:** ~57 minutes  
**Date:** July 27, 2026  

---

## 🎯 Executive Summary (Core Essence & Detailed Takeaways)

This 57-minute testing session covers key bugs, usability flaws, and architectural enhancement requests across PowerX Keys V2:

### 1. 🐛 Primary UI & Gear Menu Bugs
* **App Scope Settings:** In `My Macros` card gear menu -> `App Scope` (`Include Only`), remove the unnecessary list button, fix the broken `Capture App` icon rendering, and fix the misleading hover tooltip (*"Capture Active Window 3s delay"*).
* **Gear Menu Closing & Popups:** Secondary gear menus inside `Image Pattern` cards do not close when opening the main card gear menu. Clicking `Capture UI Element` fails to trigger overlay
* **Dropdown Selection Reset:** In `If/Else` block settings (`Target Success`), selecting dropdown items (e.g. `Color 1`) gets cleared/reset when dragging the block or clicking away due to UI virtualization recycling.

### 2. 🎛️ Timeline, Block Cards & Inspection Fixes
* **Recorded Window Block Cleanup:** Recorded focus window blocks show editable property buttons (List, Pinpoint, Gear) on timeline cards. Recorded blocks should NOT have editable properties.
* **Block Label Naming:** Timeline block headers display long window titles (`Testing Checklist - Obsidian Vault... . Obsidian`) as primary bold text instead of clean short App Names (`Obsidian`).
* **UI Element & Image Capture Fixes:** Preview image thumbnails are missing/blank on timeline block cards after capture. `Check Exists` condition in `UI Element` blocks incorrectly evaluates to `No` even when elements exist, and preview highlights draw misplaced red boxes when elements are missing.
* **Search Area Overlay:** Capturing a custom search area leaves the button label stuck as *"Smart Search Area"*, and clicking overlay guide text during selection interrupts the drag gesture.

### 3. 🖱️ Multi-Select & Drag-and-Drop Operations
* **Multi-Select Movement & Deletion:** Multi-selecting multiple timeline blocks and dragging to the top, right-clicking to Delete, or using `Ctrl+Up`/`Ctrl+Down` only operates on 1 block instead of all selected items.
* **Nested Container Drag Lines:** Dragging blocks inside deeply nested `If/Else` containers (level 2 / level 3) causes insertion guide lines to stretch out of bounds across parent containers.

### 4. 🛠️ App State, Recording & Licensing
* **Focus Window Naming:** Recording window focus changes records raw system process strings like `program_manager.explorer` instead of clean user labels like `Desktop`.
* **Alt+F4 Interception:** Pressing `Alt + F4` during recording breaks app responsiveness.
* **Offline Sign-in Loop:** Reopening the app offline forces users back to the Welcome/Sign-in screen even with valid saved tokens.

### 5. 💡 Usability Enhancements & Enhancements
* **Visual Polish & Validation:** Align `Pixel Color` and `Image Pattern` gear icon container styling, add a visual badge directly on Mouse block cards when *"Human Flow"* is enabled, and clean up vertical 2px side margin lines inside container branches.
* **Error Tooltips & Empty Loop Guard:** Provide specific error tooltips (e.g., *"Pixel 'Color 1' Not Found in Block #3"*) instead of generic alerts, and disable Save/Preview on empty `If/Else` or `Repeat` blocks with a warning badge.
* **Timeline Performance:** Add skeleton loading placeholders while populating timeline blocks after recording to prevent blank timeline flickers.

### 6. ✨ Architectural & Feature Roadmap
* **Trigger & Smart Recorder Upgrades:** Plan a clean split for Press & Release triggers under `Hold & Release`. Auto-collapse raw window focus + click + minimize sequences into single *"Minimize Window"* blocks, and auto-merge consecutive `Key Down` / `Key Up` events for the same key into a single `Key Press` block.
* [x] ~~**Negative Search Optimization:** Skip 4-level image cascade fallbacks when evaluating negative conditions (checking IF image is *NOT* present).~~ *(Evaluated & Resolved: Kept system as-is; current cascade & Smart Box cache handle searches reliably without code changes)*
* **Saved Templates & Kill Switch:** Separate full **Saved Macros** from reusable **Saved Templates**, and add `Shift + Escape` as a 1-key combo kill switch alongside `Double Escape`.

---

## 🐛 Bugs & Issues

### 1. Dashboard & App Scope
* [x] **Remove Extra List Button:** Under **My Macros** card -> **Gear Menu** -> **App Scope**, selecting `Include Only` displays a `Capture App` button and an extra list button. Remove the list button completely. *(Fixed in `ScriptLibraryView.xaml`)*
* [x] **Capture App Icon Rendering:** The icon inside the `Capture App` button is broken / not rendering properly. *(Replaced emoji ⚡ with Segoe MDL2 glyph `&#xE77B;` in `ScriptLibraryView.xaml`)*
* [x] **Misleading Tooltip:** Hovering over `Capture App` displays *"Capture Active Window 3 seconds delay"*, which is completely wrong and misleading for App Scope. Fix tooltip text. *(Updated to "Select a target application for this scope rule" in `ScriptLibraryView.xaml`)*

### 2. Gear Menus & Dropdown UI
* [x] **Nested Gear Menu Close Bug:** In `Image Pattern` mode, clicking the main card gear menu while the secondary/nested `Capture Image` gear menu is open does not close the secondary menu. It only closes when clicking empty dashboard space. *(Added visual-tree walk in `SettingsPopup_Opened` in `TemplateHandlers.cs` to close sibling gear popups)*
* [x] **UI Element Trigger Capture Button:** Clicking the `Capture UI Element` button is broken or doesn't trigger/open properly. *(Fixed — added exception handling, hook failure detection, 60s timeout, and re-entrancy guard in `UIElementCaptureService.cs` + `MacroEditorViewModel`)*
* [x] **Toggle A/B Mode Broken Icon:** In Trigger Mode `Toggle A/B`, the icon next to *"Second Press"* is broken / displaying gibberish characters. *(Fixed em dash and glyph rendering in `CustomActionCard.xaml`)*
* [x] **If/Else Drag/Drop Reset Bug:** In If/Else block settings, selecting *"Target Success"* shows a Name dropdown (e.g. `Color 1`). Clicking and dragging `Color 1` or the block causes the dropdown selection to clear/reset automatically. *(Root cause: `VirtualizingPanel.VirtualizationMode="Recycling"` on branch ItemsControls. Fixed by disabling virtualization in `MacroStepCard.xaml`)*
* [x] **Context Menu Misplacement (Mouse Coordinates):** Right-clicking non-mouse blocks (or mouse blocks where coordinates don't apply, e.g. Scroll Up/Down) still shows the *"Edit Coordinates"* context menu option. *(Collapsed via DataTrigger in `MacroEditorView.xaml` for WheelUp/WheelDown steps)*
* [x] **Context Menu Clear Bug (If/Else):** When editing `If/Else` -> `Target Success` -> selecting a dropdown item (e.g. `Color 1`), dragging the block or clicking away clears the dropdown selection. *(Same root cause and fix as If/Else Drag/Drop Reset Bug above — virtualization recycling)*

### 3. Timeline & Block Cards
* [x] **Recorded Window Blocks showing Editable Properties:** Recorded window focus blocks display property buttons (List, Pinpoint, Gear icon) on the timeline block card. Recorded blocks should NOT show editable properties. *(Collapsed List/Pinpoint/Gear buttons when `IsManuallyAdded=False` via DataTrigger in `MiscTemplates.xaml`)*
* [x] **Block Label Title Overwrite:** Timeline blocks show the long window title (e.g., `Testing Checklist - Obsidian Vault... . Obsidian`) as primary bold text instead of the short App Name (`Obsidian`). It should display `App Name` first in bold, followed by `. Title` or tooltip preview. *(Changed `StepName` from raw window title to cleaned exe name in `MacroRecordingService.cs`)*
* [x] **Captured Thumbnail Preview Image Missing:** After capturing an image or UI element, the preview image thumbnail on the timeline block card is missing/blank. *(Added screenshot capture to `UIElementCaptureService.BuildResult()` and wired in `CaptureUIElementAsync`)*
* [x] **UI Element "Check Exists" Failure:** Setting UI Element trigger to *"Check Exists"* and evaluating via If/Else always routes to the `No` (failed) branch even when the element exists on screen. *(Added relaxed UIA retry with AutomationId+ControlType matching + point-based fallback, and fixed missing `_stepSuccessStates[cleanName] = true` on fallback-success path in `MacroExecutionService.cs`)*
* [x] **Preview Box Red Highlight Misplacement:** Right-clicking a UI Element block and clicking Preview when the element is NOT on screen draws a misleading red box at a random screen location instead of remaining hidden/showing not found. *(Fixed DPI correction condition from `dpiScale != 1.0` to `dpiScale > 0` in `UIElementHighlightWindow.cs`)*

### 4. Search Area Overlay & Selection
* [x] **Smart Search Area Button Label:** Capturing a custom search area updates the area coordinates, but the button text remains *"Smart Search Area"* instead of updating to *"Custom Search Area"*. Clicking *"Full Screen"* updates area, but clicking *"Use Search Area"* reverts without updating label. Button text should dynamically reflect active mode (`Smart Search`, `Custom Search`, `Full Screen`). *(Added `OnPropertyChanged(nameof(SearchScopeDisplay))` to `X`, `Y`, `SearchWidth`, `SearchHeight` setters in `MacroItem.cs`)*
* [x] **Overlay Guide Text Interception:** During click & drag search area selection, clicking on the overlay guide text ("Use Smart Area") intercepts the selection gesture — guide text should ignore mouse clicks during selection. *(Set `IsHitTestVisible="False"` on `GuideTitleIcon`, `GuideTitle`, `GuideSubDesc`, `CollapsedIcon` in `CaptureOverlay.xaml`)*

### 5. Multi-Select & Drag-and-Drop
* [x] **Multi-Select Top Drag Failure:** Multi-selecting multiple blocks and dragging them to the top of the timeline only moves 1 block instead of all selected blocks. *(Added auto-expand timer `_expandOnDragTimer` in `MacroEditorView.xaml.cs` + `TimelineListBox_DragOver` logic in `MacroEditorView.DragDrop.cs` to expand collapsed containers on hover)*
* [x] **Multi-Select Delete Failure:** Multi-selecting blocks and right-clicking -> Delete only deletes 1 block. *(Added `Tag="DeleteStepMenuItem"` + code in `OnTimelineContextMenuOpening` to switch between `RemoveStepCommand` and `BulkDeleteCommand` based on selection count)*
* [x] **Multi-Select Shortcut Move Failure:** Multi-selecting blocks and using `Ctrl + Up` / `Ctrl + Down` only moves 1 block. *(Changed `MoveUp`/`MoveDown` handler to iterate all `SelectedItems` in correct order — top-to-bottom for up, bottom-to-top for down — in `MacroEditorView.Events.cs`)*
* [x] **Nested Container Drag-and-Drop Erratic Behavior:** Dragging blocks inside deeply nested `If/Else` blocks (level 2 / level 3) causes indicator lines to stretch out of bounds across parent containers and places blocks incorrectly. *(Fixed depth offset double-counting in `MacroEditorView.DragDrop.cs:637` — `element.Depth` already includes branch nesting, removed extra `+16`. When `element==null`, depth is now derived from parent LogicIf step)*

### 6. Recorder & App State
* [x] **Program Manager Raw Window Name:** Recording focus window changes displays raw system process names (e.g., `program_manager.explorer`) instead of clean display names (e.g., `Desktop` or `File Explorer`). *(Addressed by 3b fix — `StepName` changed from raw window title to cleaned exe name. Also fixed `StepName` collision using variable names with unique IDs in `MacroRecordingService.cs`)*
* [x] **Alt+F4 During Recording Glitch:** Pressing `Alt + F4` while recording is active stops recording but leaves the app tray/timeline in an unresponsive state. *(Added `PreviewKeyDown="Window_PreviewKeyDown"` for Alt+F4 detection + `Closing` event handler to stop recording when widget is forcefully closed)*
* [x] **Net Disconnect Sign-in Loop:** Closing/reopening the app while offline forces the user back to the *"Welcome / Sign In"* screen even if previously logged in. *(Three-part fix: network vs auth error detection in `SupabaseAuthService.cs`; skip auth when saved tokens exist; subscription cache with 7-day TTL and 3-day grace fallback)*

---

## 💡 Improvements & Enhancements

### 1. UI Consistency & Visual Polish
* [x] **Gear Icon Container Consistency:** On `Pixel Color` blocks, the Gear icon has a background container, whereas on `Image Pattern` blocks it does not. Both should share identical UI container styling. *(Fixed in `CustomActionCard.xaml` — `ImageSettingsToggle` now matches `PixelSettingsToggle` styling)*
* [x] **Human Flow Visual Badge:** Toggling *"Human Flow"* on a Mouse block turns the context menu toggle green, but no visual badge/indicator appears directly on the timeline block card to indicate Human Flow is enabled. *(Added `HumanFlowBadge` in `MacroStepCard.xaml` — visible via MultiDataTrigger on Type=MouseClick + UseHumanization=true)*
* [x] **Side Margin Lines in Containers:** Remove or refine the 2px vertical gray guide lines on the left side of `If/Else` `Yes` / `No` container branches. *(Removed `BorderThickness="2,0,0,0"` from both `ChildStepsContainer` and `ChildStepsFalseContainer` in `MacroStepCard.xaml`)*
* [x] **Flicker-Free Undo/Redo:** Undo/Redo on complex nested blocks (`Groups`, `If/Else`, `Repeat`) causes full timeline UI redraw/flicker. Optimize timeline rendering to prevent flicker. *(Replaced `Clear()`+`Add()` with atomic collection replacement, removed redundant `TimelineRebuildRequested` in `MacroEditorViewModel.Commands.cs`)*

### 2. Validation & Tooltips
* [x] **Specific Error Tooltips:** Replace generic error tooltips (e.g. *"Pixel Not Found"*) with detailed messages specifying the exact block (e.g., *"Pixel 'Color 1' Not Found in Block #3"*). *(Added `ValidationMessage` property on `MacroStep` with step-name prefix and type-specific messages, bound to WarningDot tooltip in `MacroStepCard.xaml`)*
* [x] **Empty Block Validation:** Empty `If/Else` or `Repeat` blocks (0 nested actions) or `Repeat` count = 1 should gray out Save/Preview buttons and display a warning badge (*"Empty Loop"* / *"Invalid Condition"*). *(Added empty-container checks to `IsValid` for LogicIf and LoopSequence, added `ValidationMessage` in `MacroItem.cs`)*
* [x] **Trial Expiration Screen Polish:** On trial expired modal, hide the *"Log Out / Switch Account"* link to prevent trial bypass attempts, leaving only *"Upgrade"* and *"Exit"*. *(Removed `LogOutButton` from `SubscriptionExpiredWindow.xaml` and its handler from code-behind)*

### 3. Timeline Performance
* [x] **Skeleton Loading Placeholder:** When populating timeline blocks after recording or loading, display animated skeleton placeholder cards instead of leaving the timeline blank for 1–2 seconds. *(Already exists — post-recording skeleton at lines 1697–1833 and editor skeleton at lines 1995–2227 in `MacroEditorView.xaml`)*

---

## ✨ New Feature Ideas & Architecture

### 1. Trigger Mode Press & Release Split
* [x] **Split Hold/Press & Release:** Removed combined `PressAndRelease` mode from UI + compiler. Exposed `Release` (Key Up) as a separate trigger mode in the ComboBox. Users now create two independent action cards: one with `Press (Key Down)` and one with `Release (Key Up)` on the same key. *(Removed PressAndRelease ComboBoxItem, pill styling, compiler blocks, `ReleasePath` property. Added "Release (Key Up)" ComboBoxItem for `TriggerMode.Release`)*

### 2. Smart Recording Optimizations
* [x] **Key Press Auto-Merge:** Real-time merge of KeyDown+KeyUp into a single `Press` step when no delay step exists between them (quick tap). Implemented in `MacroEditorViewModel.Recording.cs` callback. *(Guarded by `pendingDelay == null` — when a delay step was emitted between KeyDown and KeyUp due to diff >= 200ms threshold, the merge is skipped, preserving held-key semantics.)*
* [x] **Window Action Collapse:** Post-recognition of Minimize (click → focus change), Close (click → window destroyed), and Maximize (click → WindowAction Move) patterns. Collapse into single smart `WindowAction` blocks. Implemented in `CollapseWindowActions()` in `MacroEditorViewModel.Recording.cs`, called after `cleanAndOptimize`. *(Uses window geometry from preceding `WindowAction(Activate)` to compute caption button regions — ~45px per button, detect by X coordinate ranges near right edge.)*

#### A. Window Action Collapse — Plan

**Current behavior:** Recording only captures `WindowAction (Activate)` for focus changes and `WindowAction (Move)` for move/resize. Minimize, Maximize, Close, Restore are NOT auto-detected — user must add them manually.

**Goal:** During recording (or post-recording cleanup), detect the actual window operation performed by the user and collapse raw steps into a single smart `WindowAction` block with the correct `Value`.

**Approach:** Post-recording post-processing (scan recorded steps after recording stops and collapse detected patterns). This is preferred over real-time detection because we need full context (e.g., did the window actually minimize or just lose focus briefly?).

---

**1. Window Minimize Detection**

- **Raw pattern:** MouseClick on minimize button area (top-right `─` button coordinates of foreground window) → within ~500ms, a `WindowAction (Activate)` fires for a *different* window (the window behind or desktop) → the original window is no longer foreground.
- **Collapse logic:** If a MouseClick at coordinates matching the current window's minimize button region is followed by a focus switch to a different window/desktop within a short window, replace those steps with a single `WindowAction { Value = "Minimize", WindowTitle = originalWindow }`.
- **Edge cases:**
  - Window was already minimized via keyboard (`Alt+Space`, `N`): No MouseClick captured. Detect based on `KeyDown(Alt) → KeyPress(Space) → KeyPress(N)` pattern instead. Rare — can be deferred to v2.
  - Snap/Layout: Windows 11 snap layouts show a popup on hover — don't trigger minimize detection on the hover click; only on actual click-and-minimize.

---

**2. Window Close Detection**

- **Raw pattern:** MouseClick on close button area (top-right `✕` button) → within ~500ms, a `WindowAction (Activate)` fires for a *different* window → the original window is no longer in the alt-tab list (window destroyed).
- **Collapse logic:** Same as minimize but targeting the close button region. Replace with `WindowAction { Value = "Close", WindowTitle = originalWindow }`.
- **Edge cases:**
  - Close via `Alt+F4`: Detect `KeyDown(Alt) + KeyPress(F4)` pattern. Emit `Close` block.
  - Close via middle-click on taskbar thumbnail: Harder to detect (no click coordinates map to a close button). Defer to v2.
  - Document tab close (not window): Should not trigger — need to verify the window actually disappeared, not just a tab. Use HWND validity check via Win32 `IsWindow()`.

---

**3. Window Maximize Detection**

- **Raw pattern:** MouseClick on maximize button area (top-right `□` button) → within ~500ms, a `WindowAction (Move)` fires with new size matching near-full-screen dimensions (or a snap quadrant) → same window remains focused.
- **Collapse logic:** Merge MouseClick + subsequent WindowAction(Move) into `WindowAction { Value = "Maximize", WindowTitle, WindowX/Y/Width/Height = new values }`.
- **Edge cases:**
  - Maximize via double-click on title bar: No MouseClick on maximize button. Instead, a MouseClick (double-click) on the title bar area. Detect `ClickCount == 2` on title bar coordinates. Emit `Maximize`.
  - Maximize via `Win+Up`: `KeyDown(Win) + KeyPress(Up)`. Hard to distinguish from snap. Defer keyboard shortcut detection to v2.
  - Snap to half/quadrant: The Move event has specific dimensions (half screen). Should this be `Maximize` or `Snap Left`? Decision: Emit `Maximize` for true full-screen; for partial snap, keep as `Move` with the snap dimensions shown in the step summary.

---

**4. Window Restore Detection**

- **Raw pattern:** From maximized state → MouseClick on maximize button (which now shows `❐` restore icon) → window shrinks to previous size → `WindowAction (Move)` fires with restored dimensions.
- **Collapse logic:** Merge into `WindowAction { Value = "Restore", WindowTitle }`.
- **Edge case:** Restore via dragging title bar of maximized window down → MouseDown on title bar → MouseTrace downward → MouseUp → Move event at non-maximized position. Detect by checking: was the window maximized before the drag? Tricky without tracking window state. Defer to v2.

---

**5. Window Move Detection (already partially handled)**

- **Current:** WinEvent hook already emits `WindowAction (Move)` after 600ms debounce.
- **Enhancement:** If the Move is preceded by a MouseDown → MouseTrace on the title bar area, collapse the trace steps out and keep only the Move block (the trace is noise for a simple drag). The final step should show `"Move Window"` with position delta.
- **Edge case:** If the user moves and then clicks something in the new position, the MouseTrace is meaningful — don't collapse. Only collapse when the trace ends without a subsequent click/action.

---

**6. Window Resize Detection**

- **Raw pattern:** MouseDown on window edge/border → MouseTrace (dragging to resize) → MouseUp → `WindowAction (Move)` fires with new dimensions.
- **Collapse logic:** Detect that the MouseDown coordinates are on the window's resize border (left/right/top/bottom edges within 5px). Merge the trace + Move into `WindowAction { Value = "Resize", WindowTitle, WindowWidth/Height = new dimensions }`.
- **Edge case:** Resize from corner — coordinates are on both an edge and a corner. Still counts as resize.

---

**7. Combined Move + Resize (Drag to Snap)**

- **Raw pattern:** MouseDown on title bar → drag to screen edge → snap animation → both position AND size change in the Move event.
- **Collapse logic:** If a single Move event changes both position and size significantly (window wasn't maximized before), emit `WindowAction { Value = "MoveResize" }` with all values.
- **Alternative:** Keep as `Move` since the user primarily intended to move, and the resize was a side effect of snapping. The step summary can show both position and size: `"Moved → (X, Y)  Width×Height"`.

---

**Implementation:** Post-recording `CollapseWindowActions()` method added in `MacroEditorViewModel.Recording.cs`, called after `cleanAndOptimize` for both `_recordedNoTraceSteps` and `_recordedAllTraceSteps`. Detects Minimize (click on left caption button + focus switch), Close (click on rightmost caption button + focus switch), and Maximize (click on middle caption button + WindowAction(Move) on same window). Uses window geometry from preceding `WindowAction(Activate)` to compute caption button regions.

**Future phases:**
1. (done) **Phase 1 (Post-recording):** `CollapseWindowActions()` runs during post-processing.
2. **Phase 2 (Manual clean-up):** Add a "Clean Up Recording" button in the UI.
3. **Phase 3 (Real-time):** For simple cases (minimize, close), detect during recording.

---

#### B. Key Press Auto-Merge — Plan

**Current behavior:** Each KeyDown → separate `Keyboard { KeyActionType = "Hold Down" }` step. Each KeyUp → separate `Keyboard { KeyActionType = "Released Up" }` step. A single tap produces 2 timeline entries.

**Goal:** When a key is pressed and released quickly (normal typing), merge the Down + Up pair into a single `Keyboard { KeyActionType = "Press" }` step. Only keep separate Down/Up when the key is held for longer than a threshold.

**Approach:** Real-time merge in ViewModel callback (`MacroEditorViewModel.Recording.cs`). When a "Released Up" arrives with no buffered delay (meaning no pause between KeyDown and KeyUp), check if the last step in `_targetCollection` is a matching "Hold Down". If so, change that step's `KeyActionType` to "Press" and discard the "Released Up" step. Leverages existing `CheckDelay()` behavior — when diff >= threshold (200ms), a delay step is emitted before KeyUp, which buffers in `pendingDelay` and prevents the merge.

**Implementation:** `MacroEditorViewModel.Recording.cs:580-595` — inserted before pending delay flush, guarded by `pendingDelay == null`.

---

### 3. Image Cascade Search Optimization
* [x] ~~**Negative Condition Search Optimization:** When evaluating IF image is *NOT* present, running all 4 cascade fallback levels wastes execution time. Explore logic/options to skip cascade fallbacks when checking negative conditions.~~ *(CLOSED — Evaluated & Resolved: Kept as-is after multi-agent review; zero code changes needed)*

### 4. Macro vs Template Architecture
* **Saved Templates vs Saved Macros:** Introduce a clear architectural distinction between full **Saved Macros** and reusable **Saved Templates** (block logic snippets like custom If/Else constructs), updating *"Call Macro"* block accordingly.
* **Template Creation & Insertion Workflow:**
  * **Creation:** Multi-select blocks on timeline ➔ Right-click ➔ *"Save as Template"* ➔ Assign name ➔ Save.
  * **Insertion:** Add `Insert Template` block (starts empty) ➔ Select template from dropdown ➔ Edit trigger appears once assigned.
* **Template Editing & Trigger Experiments:**
  * **Option 1 (Hover Pencil Icon):** Pencil icon `✏️` appears on hover over the block card for instant 1-click editing.
  * **Option 2 (Right-Click Context Menu):** Right-click block card ➔ select *"Edit Template"*.
  * **Recommendation:** Include **Pencil Icon on Hover** as primary visible shortcut + **Right-Click** in context menu so both workflows are available.
* **Navigation UX Experiments (Dummy UI Phase):**
  * *Option A (Tabs):* Browser-style top tabs (`Macros` | `Templates`).
  * *Option B (Inline View Switch):* View switches inside the same window with a `← Back` button (like navigating folders).
  * *Prototyping Strategy:* Antigravity builds both navigation styles and edit triggers in a dummy UI prototype so user can test and pick the winner before backend wiring.

### 5. Kill Switch Shortcut Update
* [x] **Add Shift + Escape Shortcut:** Changed default from `Double Escape` to `Shift + Escape`. Updated all 4 fallback `?? "Double Escape"` to `?? "Shift + Escape"` in RecordingWidgetView, MacroRecordingService, MacroExecutionService, and ScriptCompilerService.
