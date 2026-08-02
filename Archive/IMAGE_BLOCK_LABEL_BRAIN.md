# 🧠 Image Block Label Brain

> **Purpose:** Persistent memory for Image Block label feature + search cascade work.
> **Last Updated:** 2026-07-08 00:08 IST

---

## 🔄 Active Tasks

### Image Block — Source Window Label (TEMP)
- **Status:** ✅ Working
- **What it does:** Shows `AppName · WindowTitle` on the Image block row after capture
- **Temporary:** Yes — user will remove later, for testing only

### Image Search — 3-Step Cascade
- **Status:** ✅ Implemented + tested
- **What it does:** Search falls through: Smart box → Window → Full screen
- **How:** When scope is "Smart Search" and step has `SearchImageSourceAppName`, window fallback is enabled via AHK `WinExist` + `WinGetPos`
- **Finding:** When user re-adds block fresh + captures → works. Recapture without removing → sometimes fails (was due to DB persistence issue, now fixed)

### Search Cascade Checkboxes (TEMP)
- **Status:** ✅ Added to gear menu
- **What it does:** 4 checkboxes in Image block gear menu under "SEARCH CASCADE":
  - ⓪ Last Position (blue) — check cached spot
  - ① Smart Box (green) — small area around capture
  - ② Window (yellow) — whole window
  - ③ Full Screen (red) — entire screen
- **Behavior:** All OFF by default. When any is ON, only those steps run. When all OFF, normal behavior.
- **Saved to:** ExtraJson in database (survives restart)

### Image Search Lab (Sandbox App)
- **Status:** 🔨 Basic version built, testing DPI fix
- **Location:** `PowerX Keys/ImageSearchLab/`
- **Desktop shortcut:** `Image Search Lab.lnk`
- **Purpose:** Isolated sandbox to test image search engine without breaking main app
- **Features:**
  - Capture Region (drag-to-select)
  - Load Image from file
  - Search with FindText or Legacy engine
  - Tolerance sliders (Fg, Bg, Legacy)
  - Full Screen / Smart Box scope
  - Detailed logging to `sandbox_log.txt` (auto-clears on restart)
- **Known issues:** DPI offset on capture overlay (being fixed)

---

## ❌ Failed Approaches (Don't Repeat!)

| Attempt | Why It Failed |
|---------|--------------|
| TextBlock in `SummaryContainer` (MacroStepCard.xaml) | Collapses when settings panel is open |
| `WindowFromPoint` inside overlay | Detects the overlay itself |
| `_prevFocusedHwnd` | Shows wrong window (focused before capture button click) |
| `_lastHighlightedHwnd` | Always zero in sticky mode |
| `this.Opacity = 0` before `WindowFromPoint` | WPF doesn't propagate to Win32 fast enough |
| `this.Visibility = Visibility.Hidden` | Broke image capture entirely |
| Setting scope to `WIN_SMART:` directly for sticky mode | Skips smart box, goes straight to window search |
| `SearchImageSourceAppName` without DB persistence | Values lost on auto-save/restart → no window fallback |
| `SystemParameters.VirtualScreen*` for capture | Returns DPI-scaled logical pixels → screenshot offset |

## ✅ Working Approach
- **Drag mode:** Overlay parses `CapturedScope` for app name + title
- **Sticky mode:** ViewModel does `WindowFromPoint` AFTER overlay closes
- **Label location:** `SearchTemplates.xaml` → `ImagePanelInlineTemplate`, right after "Image" text
- **Search cascade:** `ScriptCompilerService.SingleStep.cs` — enabled `hasWindowFallback` for Smart Search when `SearchImageSourceAppName` is set
- **DB persistence:** `SearchImageSourceAppName` + `SearchImageSourceWindowTitle` saved/loaded via ExtraJson
- **Screen capture:** Use Win32 `GetSystemMetrics` for physical pixel dimensions

---

## 📋 Future Features

### Option: Disable Full Screen Fallback
- **When ON:** Search cascade = Smart box → Window only (no full screen)
- **When OFF:** Search cascade = Smart box → Window → Full screen
- **Location:** Either gear menu on Image block OR global settings
- **Status:** 🔮 Future

### Option: Disable Last Position Check (Step 0)
- **Currently:** No toggle exists — last position check always runs
- **Location:** Gear menu or global settings (TBD)
- **Status:** 🔮 Future

### Optimize Main App Capture Overlay Smoothness
- **Problem:** Main app's sticky box is noticeably less smooth than the sandbox app
- **Cause:** Main app has heavy UI, macro engine, database running in background
- **Goal:** Apply same optimizations from sandbox overlay to main app's CaptureOverlay
- **Status:** 🔮 Future

### Full Search Cascade (for reference)
0. Last position (cached spot) — instant
1. Smart box — small area around capture point
2. Window — whole window
3. Full screen — everywhere

---

## 📚 Key Technical Learnings

### Files Changed (Main App)
- `CaptureOverlay.xaml.cs` — `CapturedProcessName`/`CapturedWindowTitle` + logging
- `MacroEditorViewModel.Capture.cs` — Win32 imports + post-overlay WindowFromPoint fallback
- `MacroItem.cs` — `SearchImageSourceAppName`, `SearchImageSourceWindowTitle`, `SearchImageSourceLabel`, 4x `SearchStep*` toggles (removed JsonIgnore from source props)
- `SearchTemplates.xaml` — Temp TextBlock + 4 cascade checkboxes in gear popup
- `ScriptCompilerService.SingleStep.cs` — Window fallback for Smart Search scope + cascade checkbox overrides
- `MacroDatabase.cs` — Save/load ExtraJson for source window info + cascade toggles
- `App.xaml.cs` — Auto-clear `ImageSearch_Diagnostic.log` on startup

### Files Created (Sandbox)
- `ImageSearchLab/MainWindow.xaml` + `.cs` — Dark themed UI with capture, search, tolerance controls
- `ImageSearchLab/CaptureOverlay.xaml` + `.cs` — Region selection overlay with DPI handling
- `ImageSearchLab/App.xaml` + `.cs` — App startup with log auto-clear

### Architecture
- `_prevFocusedHwnd` = window focused BEFORE overlay opened
- `_lastHighlightedHwnd` = only set in Smart/Window modes, not sticky
- `CapturedScope` = only set in drag mode, not sticky mode
- Sticky mode = single click to pin, second click to capture
- Label template lives in `SearchTemplates.xaml` → `ImagePanelInlineTemplate`
- `SummaryContainer` collapses when ConfigContainer is visible

### Key Discovery
- **Auto-save kills in-memory values:** After capture, macro auto-saves to DB. Properties not in ExtraJson are lost → must persist everything needed for search.
- **DPI scaling:** WPF `SystemParameters` returns logical pixels. Use Win32 `GetSystemMetrics` for physical pixels when doing screen capture.
- **Image search 50/50 issue:** Not a scope/cascade bug — the actual FindText engine sometimes fails to match even when image is clearly visible. Sandbox app created to investigate root cause.

### Build & Testing
- User restarts app after every rebuild
- Main app log: `bin\Debug\net10.0-windows\ImageSearch_Diagnostic.log` (auto-clears on restart)
- Sandbox log: `ImageSearchLab\bin\Debug\net10.0-windows\sandbox_log.txt` (auto-clears on restart)
- `LogDiagnostic()` writes timestamped entries
