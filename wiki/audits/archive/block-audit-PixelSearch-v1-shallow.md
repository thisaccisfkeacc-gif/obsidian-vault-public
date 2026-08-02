# Block Audit: PixelSearch

## Summary
PixelSearch finds a specific hex color within a search region. Has a unique 2px self-healing fallback for subpixel rendering shifts. Shares scope parsing with ImageSearch.

---

### [SEVERITY: Low] — No window fallback (unlike ImageSearch)
**Scenario:** PixelSearch with WIN_LIVE/WIN_REL scope fails in the primary area.
**Impact:** Unlike ImageSearch which has a `hasWindowFallback` second-chance search over the full window, PixelSearch only has the 2px self-healing expansion. If a pixel color shifts due to window repositioning beyond 2px, it fails.
**Verified:** Yes — no `hasWindowFallback` code path exists for PixelSearch.
**Fixed:** No — by design. PixelSearch targets are typically stationary (status LEDs, icons). The 2px self-healing covers subpixel rendering. A full-window fallback would be too slow and prone to false positives (finding the same color elsewhere).

### [SEVERITY: Low] — Hex color sanitization strips to last 6 chars
**Scenario:** User enters an 8-character hex (e.g., "FF00FF00" with alpha) or malformed hex.
**Impact:** Code: `if (hex.Length > 6) hex = hex.Substring(hex.Length - 6)` — takes the LAST 6 chars. If input is "FFRRGGBB" (8-char with alpha prefix), this correctly extracts "RRGGBB". If input is garbage, it'll just produce a non-matching color (graceful failure, not a crash).
**Verified:** Yes
**Fixed:** N/A (correct behavior)

### [SEVERITY: Low] — Tolerance clamped 0-255 with Math.Clamp
**Scenario:** User somehow sets tolerance to 300 or -5.
**Impact:** `int clampedTolerance = Math.Clamp(step.Tolerance, 0, 255)` — safely bounded. AHK's PixelSearch tolerance parameter is 0-255.
**Verified:** Yes
**Fixed:** N/A

### [SEVERITY: Low] — Self-healing 2px expansion uses Max(0, ...) for left/top only
**Scenario:** Pixel is near screen top-left corner (e.g., X=0, Y=1).
**Impact:** `Max(0, ({x1}) - 2)` prevents negative X1/Y1. But X2/Y2 expansion (`({x2}) + 2`) could exceed screen bounds. AHK's PixelSearch handles out-of-bounds coordinates gracefully — it simply clips to the actual screen dimensions.
**Verified:** Yes — AHK v2 PixelSearch docs confirm coordinates beyond screen bounds are automatically clipped.
**Fixed:** N/A

### [SEVERITY: Low] — Debug highlight uses animated ripple (expensive)
**Scenario:** User enables debug highlight on a PixelSearch block.
**Impact:** 30-frame animation loop with `Sleep 40` = 1.2 seconds of animation blocking. During this time, the macro is paused. This is only for debugging, not production use. The dot overlay and ripple are separate GUIs both destroyed after animation.
**Verified:** Yes
**Fixed:** N/A (debug feature, expected to be slow)

### [SEVERITY: Low] — WIN_SMART scope treated as WIN_REL for PixelSearch
**Scenario:** SearchScopeSummary starts with "WIN_SMART:".
**Impact:** Code: `bool isRel = step.SearchScopeSummary.StartsWith("WIN_REL:") || step.SearchScopeSummary.StartsWith("WIN_SMART:")` — WIN_SMART is treated as relative. This is correct because WIN_SMART means "smart window-relative" coordinates.
**Verified:** Yes
**Fixed:** N/A

---

## Verdict

✓ No actionable issues found. PixelSearch is simpler than ImageSearch but equally robust. The 2px self-healing fallback is a smart addition for subpixel drift.
