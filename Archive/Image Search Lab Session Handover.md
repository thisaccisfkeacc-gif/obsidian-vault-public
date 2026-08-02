

## Work Completed & Solved

1. **FindText/Turbo Fix:** Fixed code generator in sandbox to match main app robust encoding.
2. **Legacy ImageSearch DPI Fix:** Added screen CoordMode and DPI awareness.
3. **Capture Overlay Sticky Mode:** Aligned sizing/crop coordinates to physical pixels.
4. **UI Improvements:** Replaced sliders with one dark-themed preset dropdown.
5. **Highlighting:** Added transparent overlay that highlights found matches for 3 seconds.
6. **v2 Upgrade:** Complete rewrite of sandbox UI — side-by-side engine comparison, batch 10x reliability test, simplified dark UI.
7. **2 Tolerance Dropdowns:** Separate presets for FindText (Fg/Bg) and Legacy (*var), 5 options each.
8. **Capture Overlay Dual Mode:** Sticky box (scroll resize, click-pin, click-confirm) + click-and-drag custom area.
9. **Legacy Multi-Match Loop:** AHK loop that finds ALL instances (shifts search area after each find, capped at 50).
10. **Cascade (Auto) Scope:** 4-step fallback — Last Position → Smart Box → Window → Full Screen.
11. **Auto Window Detection:** Uses `WindowFromPoint` at capture center to get source window bounds for cascade step 3.
12. **Last Position Cache:** Auto-stores position after 2 consecutive finds at same spot (8px padding).
13. **Individual Engine Buttons:** "Test FindText" and "Test Legacy" buttons for isolated testing.

## Current Issues (Unresolved)

1. **Sticky mode second-click not capturing:** `Mouse.Capture(SelectionCanvas)` was eating the second click event. Fix applied (removed `Mouse.Capture` from MouseDown) but needs testing.
2. **Legacy multi-match loop hanging:** Loop sometimes doesn't exit cleanly causing 5s timeout. Added `Max(imgW, 1)` step guard but fundamentally `ImageSearch` is single-match. FindText finds multiple natively, Legacy struggles.
3. **Legacy only finds 1 match:** Even with the loop, Legacy `ImageSearch` isn't reliably finding more than 1 instance. FindText finds 2+ in the same scenario. This is likely a core limitation of AHK's `ImageSearch` command.

## Next Steps

- **Test sticky capture fix** — verify second click now confirms properly
- **Decide on Legacy multi-match** — accept 1-match limitation or try alternative loop strategy (shift by 1px on Y instead of imgH)
- **Test cascade mode** — verify all 4 steps fire correctly and cache builds after 2 finds
- **Port improvements back** to main PowerX Keys app once sandbox testing is stable

# Image Block Label Brain

**Purpose:** Persistent memory for Image Block label feature + search cascade work.

**Last Updated:** 2026-07-16 (Session with Kiro)

## Active Tasks

### Image Search Lab v2

- **Status:** Upgraded, needs testing
- **Location:** `PowerX Keys/ImageSearchLab/`
- **Desktop shortcut:** `Image Search Lab.lnk`
- **Features:**
    - Side-by-side engine comparison (FindText vs Legacy)
    - Batch 10x reliability test with % scores
    - Individual "Test FindText" / "Test Legacy" buttons
    - 2 tolerance dropdowns (FindText presets + Legacy presets)
    - 3 scope options: Full Screen, Smart Box, Cascade (Auto)
    - Smart Box visual preview (highlight overlay for 2s when selected)
    - Cascade: Last Pos → Smart Box → Window → Full Screen
    - Auto window detection at capture time via `WindowFromPoint`
    - Capture overlay: sticky box + click-and-drag dual mode

### Image Block — Source Window Label

- **Status:** Working
- Shows `AppName · WindowTitle` on Image block row after capture

### Image Search — 4-Step Cascade

- **Status:** Implemented in sandbox, untested
- Last Position (cached after 2 consecutive finds) → Smart Box (60px pad) → Window (auto-detected) → Full Screen
- Main app version in `ScriptCompilerService.SingleStep.cs` uses same logic

### Search Cascade Checkboxes (Main App)

- **Status:** Added to gear menu
- Gear menu has: Last Position, Smart Box, Window, Full Screen
- Saved to ExtraJson

## Failed Approaches — Don't Repeat

|Attempt|Why it failed|
|---|---|
|TextBlock in `SummaryContainer`|Collapses when settings panel opens|
|`WindowFromPoint` inside overlay|Detects overlay itself|
|`_prevFocusedHwnd`|Shows wrong window|
|`_lastHighlightedHwnd`|Always zero in sticky mode|
|`this.Opacity = 0` before `WindowFromPoint`|WPF doesn't propagate fast enough|
|`this.Visibility = Hidden`|Broke image capture|
|Direct `WIN_SMART:` scope|Skips smart box|
|Source app name without DB persistence|Values lost after auto-save/restart|
|`SystemParameters.VirtualScreen*`|Returns logical DPI-scaled pixels|
|`Mouse.Capture(SelectionCanvas)` in sticky MouseDown|Eats the second click, prevents confirm|
|Legacy ImageSearch multi-match via simple X-shift loop|Hangs or only finds 1 — AHK limitation|

## Working Approach

- Drag mode parses `CapturedScope` for app name + title
- Sticky mode uses `WindowFromPoint` after overlay closes
- Label lives in `SearchTemplates.xaml` → `ImagePanelInlineTemplate`
- Search cascade lives in `ScriptCompilerService.SingleStep.cs`
- Source window info + cascade toggles save/load through ExtraJson
- Screen capture uses Win32 `GetSystemMetrics`
- Sandbox window detection uses `WindowFromPoint` → `GetAncestor(GA_ROOT)` → `GetWindowRect`

## Key Technical Learnings

- Auto-save kills in-memory values if not saved to ExtraJson
- WPF `SystemParameters` returns logical pixels; use Win32 `GetSystemMetrics`
- Image search 50/50 issue may be FindText engine matching, not scope/cascade
- AHK `ImageSearch` is fundamentally single-match; FindText returns all matches natively
- `Mouse.Capture` in WPF can swallow subsequent click events on the same element
- Legacy loop needs `Max(imgW, 1)` minimum step to prevent infinite loop

## Key Files

- `PowerX Keys/ImageSearchLab/MainWindow.xaml` — UI layout
- `PowerX Keys/ImageSearchLab/MainWindow.xaml.cs` — all engine logic, cascade, batch test
- `PowerX Keys/ImageSearchLab/CaptureOverlay.xaml.cs` — sticky + drag capture
- `PowerX Keys/ImageSearchLab/HighlightOverlay.xaml.cs` — match highlighting
- `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Services/ScriptCompilerService.SingleStep.cs` — main app cascade reference

## Build & Testing

- Restart app after every rebuild
- Main app log: `bin\Debug\net10.0-windows\ImageSearch_Diagnostic.log`
- Sandbox log: `ImageSearchLab\bin\Debug\net10.0-windows\sandbox_log.txt`
