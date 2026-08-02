# Engine Spinner Edge Case Scan

Files reviewed: `MainWindow.xaml` (lines 122-161), `MainWindow.xaml.cs` (lines 700-837), `MainViewModel.cs` (lines 70-215)

**Behavior:** Spinner (dashed rotating ring) appears next to START button when `IsEngineTransitioning` is true. Controlled via DataTrigger + forever animation (0.8s per rotation).

## Findings

### Medium
1. **`StartEngineOverlayTimer` is dead code** – Defined at line 820, never called. Spinner is triggered directly via `IsEngineTransitioning = true` at line 775. The timer-based path (which would delay spinner by 500ms) is unused.
2. **Stop path never shows spinner** – When user clicks STOP, `IsEngineTransitioning` is never set true. Spinner only shows during START. Stop transitions could be slow (ScriptManager.Stop takes time) with zero visual feedback.
3. **`_isProcessing` blocks ALL button clicks during transition** – `_isProcessing = true` at line 704, reset in `finally` at line 810. Any other operation that needs the button (e.g., tray icon restore + quick click) is blocked for the entire async run + 500ms.

### Low
4. **Spinner not click-blocked** – The spinner Grid has no `IsHitTestVisible="False"`. During transition, clicks on the spinner area pass through to the window (but the button itself is still reachable).
5. **`StatusBannerButton.Content` "STARTING..." / "STOPPING..." not localized** – Hardcoded English strings. Minor.
6. **`StatusBannerButton` has both `ToolTipService.ShowOnDisabled="True"` and dynamic tooltip** – Works, but the tooltip switches between conflict messages, profile-empty messages, and null. No tooltip for running state.
