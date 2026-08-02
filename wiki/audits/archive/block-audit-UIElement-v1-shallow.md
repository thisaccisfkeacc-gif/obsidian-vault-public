# Block Audit: UIElement

## Summary
UIElement uses Windows UI Automation (COM) to interact with app controls — Click, Set Text, Toggle, Select dropdowns, Read Text, Focus, Wait Until Found/Gone. Most complex single-block compilation. Supports physical click fallback, coordinate fallback, background mode, and scroll-into-view.

---

### [SEVERITY: Low] — Select action has extensive popup/overlay search (performance)
**Scenario:** User uses "Select" action on a ComboBox. The dropdown list item isn't found in the immediate element or root.
**Impact:** The code searches: (1) element's own children via SelectionItemPattern, (2) expands dropdown + searches descendants, (3) searches root element descendants, (4) searches ALL top-level windows matching the process PID or "ComboLBox"/"Popup" class names. This is thorough but could be slow (~100-500ms) on complex apps. Not a bug — just noting performance characteristics.
**Verified:** Yes
**Fixed:** N/A (correctness over speed for Select action)

### [SEVERITY: Low] — stepLabel fallback for emoji-only names
**Scenario:** User names a UIElement block with only emojis (e.g., "🔘").
**Impact:** `stepLabel = !string.IsNullOrWhiteSpace(step.StepName) ? new string(step.StepName.Where(c => char.IsLetterOrDigit(c)).ToArray()) : "UIEl"`. If all chars are stripped, `stepLabel` becomes empty string. Then: `if (string.IsNullOrWhiteSpace(stepLabel)) stepLabel = "UIEl"`. Safe fallback.
**Verified:** Yes
**Fixed:** N/A

### [SEVERITY: Low] — Wait Until Found timeout uses UITimeoutSeconds × 10 loops of 100ms
**Scenario:** UITimeoutSeconds = 10 (default).
**Impact:** `waitLoops = timeoutSec * 10` = 100 loops × 100ms = 10 seconds. The math is correct. The timeout is clamped: `Math.Clamp(step.UITimeoutSeconds > 0 ? step.UITimeoutSeconds : 10, 1, 300)`.
**Verified:** Yes
**Fixed:** N/A

### [SEVERITY: Low] — ControlClick with "NA" flag for background clicks
**Scenario:** Click without UsePhysicalClick and with a window title.
**Impact:** Uses `ControlClick("x" _RelX " y" _RelY, "ahk_id " _WinHwnd, , , , "NA")`. The "NA" flag means "Don't Activate" — the click happens without bringing the window to front. This is the correct background click pattern.
**Verified:** Yes
**Fixed:** N/A

### [SEVERITY: Low] — UseTargetOffset skips InvokePattern for Click
**Scenario:** User enables offset on a UIElement Click action.
**Impact:** When `UseTargetOffset` is true, the code skips `InvokePattern.Invoke()` and goes directly to coordinate-based click with offset applied. This is correct — InvokePattern always clicks center, so offset requires manual coordinate calculation.
**Verified:** Yes
**Fixed:** N/A

### [SEVERITY: Low] — Read Text tries ValuePattern first, falls back to CurrentName
**Scenario:** User reads text from a control that has no ValuePattern (e.g., a static Text label).
**Impact:** `try { _ValP.CurrentValue } catch { _Found.CurrentName }`. ValuePattern gives editable text (textbox content), CurrentName gives the label/title. Correct two-tier approach.
**Verified:** Yes
**Fixed:** N/A

### [SEVERITY: Low] — UIFallbackToCoordinates only for click-type actions
**Scenario:** Set Text or Toggle action fails to find element, even though X/Y coordinates are available.
**Impact:** Coordinate fallback only fires for Click/Double Click/Right Click. Other actions correctly don't fall back to coordinates (you can't "Set Text" at a coordinate). Correct design.
**Verified:** Yes
**Fixed:** N/A

### [SEVERITY: Low] — 0 conditions → _Cond := 0, FindFirst returns 0
**Scenario:** UIElement block with no identifying properties (no name, no automationId, no className).
**Impact:** `condParts.Count == 0` → `_Cond := 0` → `_Found := _Cond ? _RootEl.FindFirst(4, _Cond) : 0` → `_Found = 0`. Falls through to "not found" path. IsValid already rejects this state, so it shouldn't reach compilation.
**Verified:** Yes
**Fixed:** N/A (double-guarded by IsValid)

---

## Verdict

✓ No actionable issues found. UIElement compilation is extremely thorough with proper COM automation patterns, multiple fallback strategies, and correct AHK v2 syntax throughout. The `ComObject("{clsid}", "{iid}")` calls use the correct v2 syntax (unlike the WindowAction Desktop Click bug found earlier).
