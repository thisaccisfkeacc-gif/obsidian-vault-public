# PowerX Keys — Full Context Handoff Prompt

## Project
**PowerX Keys V2** — WPF desktop automation app (C#, .NET 10, Windows)
**Codebase:** `C:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild`

## Summary So Far
22 out of 23 bug items from `Obsidian Vault\ideas\voice_note_feedback.md` are fixed. All code compiles (DLLs build successfully). Only 1 item remains pending.

---

## All Fixes Applied (22/23)

### Section 1 — Dashboard & App Scope (All Fixed)
- **1a** — Removed extra list button in App Scope gear menu (`ScriptLibraryView.xaml`)
- **1b** — Replaced broken emoji ⚡ with Segoe MDL2 glyph `&#xE77B;` (`ScriptLibraryView.xaml`)
- **1c** — Fixed tooltip text → "Select a target application for this scope rule" (`ScriptLibraryView.xaml`)

### Section 2 — Gear Menus & Dropdown UI (1 Pending)
- **2a** — Gear popups auto-close when another opens (`TemplateHandlers.cs` — `SettingsPopup_Opened`)
- **2b** — ❌ **PENDING — UI Element capture button not activating.** Need runtime testing. Code appears correct: button binds to `CaptureUIElementCommand` which calls `CaptureUIElementAsync`. That method minimizes the window, creates overlay via `UIElementCaptureService.StartCapture()`, waits for result, and sets step properties. Likely a timing/overlay initialization issue only reproducible when running the actual app.
- **2c** — Fixed garbled em dash `C â€" 3rd Press` → `C — 3rd Press` (`CustomActionCard.xaml:693`)
- **2d** — Fixed If/Else dropdown reset on drag-drop. Root cause: `VirtualizingPanel.VirtualizationMode="Recycling"` on branch ItemsControls. Fix: removed virtualization from both YES/NO ItemsControls in `MacroStepCard.xaml`, replaced `VirtualizingStackPanel` with `StackPanel`.
- **2e** — Collapsed "Edit Coordinates" for WheelUp/WheelDown steps via DataTrigger (`MacroEditorView.xaml`)

### Section 3 — Timeline & Block Cards (All Fixed)
- **3a** — Collapsed List/Pinpoint/Gear buttons when `IsManuallyAdded=False` via DataTrigger (`MiscTemplates.xaml`)
- **3b** — Changed `StepName` from raw window title to cleaned exe name (`MacroRecordingService.cs`). Also fixed `exe`→`exeName` variable typo at line 393.
- **3c** — Added screenshot capture to `UIElementCaptureService.BuildResult()` and wired to `CaptureUIElementAsync` (`UIElementCaptureService.cs`, `MacroEditorViewModel.Capture.cs`)
- **3d** — Fixed "Check Exists" false negatives: added relaxed UIA retry (AutomationId+ControlType), point-based fallback, fixed missing `_stepSuccessStates[cleanName]=true` on fallback path (`MacroExecutionService.cs`)
- **3e** — Fixed DPI condition `dpiScale != 1.0` → `dpiScale > 0` (`UIElementHighlightWindow.cs`)

### Section 4 — Search Area Overlay (All Fixed)
- **4a** — Added `OnPropertyChanged(nameof(SearchScopeDisplay))` to X/Y/Width/Height setters (`MacroItem.cs`)
- **4b** — Set `IsHitTestVisible="False"` on GuideTitle/GuideSubDesc/CollapsedIcon (`CaptureOverlay.xaml`)

### Section 5 — Multi-Select & Drag-Drop (All Fixed)
- **5a** — Auto-expand collapsed containers on drag hover (`MacroEditorView.xaml.cs` + `DragDrop.cs`)
- **5b** — Bulk delete via right-click context menu when multiple items selected (`MacroEditorView.xaml` + `Events.cs`)
- **5c** — Multi-select Ctrl+Up/Down moves all selected items in correct order (`Events.cs`)
- **5d** — Fixed nested drag indicator depth: removed double-counting of `+16` offset when `element!=null`; when `element==null` inside container, derives depth from parent LogicIf step (`DragDrop.cs:637`)

### Section 6 — Recorder & App State (All Fixed)
- **6a** — Fixed StepName collisions with unique variable IDs (`MacroRecordingService.cs`)
- **6b** — Alt+F4 kill switch: `PreviewKeyDown` handler detects Alt+F4 and stops recording before close (`RecordingWidgetView.xaml` + `.xaml.cs`)
- **6c** — Offline sign-in loop: 3-part fix — (1) network errors don't clear saved tokens, (2) skip auth window when saved tokens exist, (3) subscription cache with 7-day TTL + 3-day grace fallback (`SupabaseAuthService.cs` + `App.xaml.cs`)

---

## What Remains

### Fix 2b: UI Element Capture Button Not Activating
**Severity:** Medium — affects UI Element trigger workflow.
**Files:** `SearchTemplates.xaml:1077` (button), `MacroEditorViewModel.Commands.cs:761-771` (command), `MacroEditorViewModel.Capture.cs:810-850` (CaptureUIElementAsync), `UIElementCaptureService.cs:85-156` (StartCapture)
**What to investigate:**
1. Open the app, add a UI Element step, click the "Capture" pill button.
2. Does the overlay window appear? Does the mouse hook install?
3. Check if `CaptureUIElementAsync` is actually being called (debugger/console log).
4. Check if `GetMainWindow()` returns null or if the window minimization interferes.
5. Check if `UIElementCaptureService.StartCapture()` encounters `_isActive == true` (previous capture not cleaned up).
6. The button uses `PreviewMouseDown="UIElementPanel_PreviewMouseDown"` on the root Border of `UIElementPanelTemplate` — verify this handler doesn't interfere with button clicks (line 513 in `TemplateHandlers.cs`).
7. Test with debugger attached to see which code path fails.

### Improvements (not bugs, not started)
The `voice_note_feedback.md` file also lists improvement items:
- Gear Icon Container Consistency
- Human Flow Visual Badge
- Side Margin Lines in Containers
- Flicker-Free Undo/Redo
- Specific Error Tooltips
- Empty Block Validation
- Trial Expiration Screen Polish
- Skeleton Loading Placeholder

These were never started — only the bug items were addressed.

---

## How to Verify Build
```powershell
cd "PowerX Keys\PowerX_Keys_V2_Rebuild"
dotnet build PowerX_Keys_V2\PowerX_Keys_V2.csproj
```
Build produces ~68 warnings (pre-existing, not related to these fixes) and 0 code errors. Occasional `MSB4166` crash is an MSBuild memory issue — just retry or build the individual projects separately.

---

## Obsidian Tracking File
Updated at: `C:\Users\Maaz\Documents\New folder\Obsidian Vault\ideas\voice_note_feedback.md`
All fixed items are marked `[x]` with fix descriptions.
