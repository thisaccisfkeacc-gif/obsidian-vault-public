# System Audit: Capture System (Deep)

**Date:** 2026-07-24
**Scope:** UIElementCaptureService, WindowCaptureService, CaptureOverlay, CoordinatePickerWindow

---

## BUGS

### 1. Multi-monitor DPI double-scaling
- **Severity:** Medium
- **Location:** CoordinatePickerWindow.xaml.cs:59-66
- **Problem:** `rawPos` already in logical units but gets multiplied by `transform.M11` again. On secondary monitors with different DPI, double-scaling occurs.

### 2. `_navLevelOffset` thread race
- **Severity:** Medium
- **Location:** UIElementCaptureService.cs
- **Problem:** `_navLevelOffset` modified by keyboard hook thread, read by UI timer tick. No synchronization. Race condition.

### 3. CaptureOverlay zero-sized capture
- **Severity:** Low
- **Location:** CaptureOverlay.xaml.cs:1317-1316
- **Problem:** Click with no prior hover produces zero-sized capture. `SaveScreenshot` guard silently returns. No user feedback.

### 4. Opacity trick unreliable for WindowFromPoint
- **Severity:** Low
- **Location:** CaptureOverlay.xaml.cs:1088-1092
- **Problem:** Setting `Opacity = 0` doesn't make overlay transparent to `WindowFromPoint`. Only `WS_EX_TRANSPARENT` does that. Works by accident via `_lastHighlightedHwnd` fallback.

### 5. Thread safety: `_frozenElement` etc.
- **Severity:** Low
- **Location:** UIElementCaptureService.cs
- **Problem:** `_navLevelOffset` not synchronized. Other state variables are UI-thread-only (safe).

---

## DEAD CODE

1. Commented-out Capture Library Auto-Save (~35 lines) — Capture.cs:272-307
2. Commented-out Pixel Library Auto-Save (~32 lines) — Capture.cs:596-628
3. Commented-out Window Library Auto-Save (~27 lines) — CaptureOverlay:1826-1853
4. `CapturedPixelWidth`/`CapturedPixelHeight` — Set but never read
5. Dead right-click handler inside left-button block — CaptureOverlay:524-527

---

## INCONSISTENCIES

1. Two completely separate capture overlay systems — UIElementCaptureService and CaptureOverlay share no code
2. Three different ESC detection patterns — Global hook, GetAsyncKeyState, KeyDown events
3. Two window picker implementations — WindowPickerWindow vs CaptureInteractiveWindowAsync
4. Inconsistent DPI handling — 4 different DPI calculation methods across capture services

---

## VERIFIED OK

- UIA property extraction — correct
- Parent container storage — correct
- Window handle resolution — correct
- Browser tab detection — correct
- Screenshot capture and Smart Box cropping — correct
- Step creation from capture — correct
- Revert on cancel — correct
