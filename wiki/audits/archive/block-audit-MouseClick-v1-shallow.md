# Block Audit: MouseClick

## Summary
MouseClick is the most complex block with 10+ action types, 6 target modes, coordinate modes, multi-click, drag/drop, and hold/release. Compilation handles all combinations.

---

### [SEVERITY: Medium] — Extra indentation on non-visible-move coordinate clicks (cosmetic, not functional)
**Scenario:** A regular click with coordinates but `IsMouseVisibleMove = false` and not inside a multi-click loop.
**Impact:** The generated AHK has `    MouseMove(...)` with extra 4-space indentation outside of any block. AHK v2 ignores leading whitespace so this executes correctly, but makes the generated script harder to read/debug.
**Verified:** Yes — lines ~2030-2033 use `$"{indent}    MouseMove(..."` outside a loop context.
**Fixed:** No — purely cosmetic, does not affect execution.

### [SEVERITY: Medium] — "No Target Selected" + "Move Mouse Only" passes IsValid check inconsistently
**Scenario:** User selects ActionTarget = "No Target Selected" and Value = "Move Mouse Only".
**Impact:** `IsValid` has: `if (ActionTarget == "No Target Selected") { if (Value == "Move Mouse Only") return false; }` — correctly rejects this combination. However, the Value dropdown in the UI might still allow selecting "Move Mouse Only" before ActionTarget is changed. The warning dot appears correctly though.
**Verified:** Yes — IsValid catches it.
**Fixed:** N/A (validation works correctly, UI shows warning)

### [SEVERITY: Low] — ScrollAmount defaults to 1 if null
**Scenario:** ScrollAmount is null (default `_scrollAmount = 1` but could be deserialized as null from old data).
**Impact:** Compilation uses `step.ScrollAmount ?? 1` — correctly defaults to 1. No issue.
**Verified:** Yes
**Fixed:** N/A

### [SEVERITY: Low] — "Active Window" target clicks center of window
**Scenario:** User uses ActionTarget = "Active Window" for a click.
**Impact:** Compiles to `(wx + ww/2)` and `(wy + wh/2)` which is the center of the active window. This is correct and documented behavior. `WinGetPos` is emitted before the click.
**Verified:** Yes
**Fixed:** N/A

### [SEVERITY: Low] — Double Click via "actionArg = 2" vs Multiple Clicks
**Scenario:** User picks a "Double" click variant (e.g., "Double Left Click" if it existed).
**Impact:** The code checks `actionStr.Contains("Double", StringComparison.OrdinalIgnoreCase)` → sets `actionArg = " 2"`. This generates `Click("Left 2")`. However, looking at the `MouseActions` array: `"Left Click", "Right Click", "Middle Click", "Multiple Clicks", "Hold Down", "Released Up", "Drag and Drop", "Right Drag and Drop", "Scroll Up", "Scroll Down"` — there is NO "Double" option in the list! So this code path is dead code that never fires.
**Verified:** Yes — the "Double" check exists in compilation but is unreachable with current UI options.
**Fixed:** No — dead code, harmless. The "Multiple Clicks" with ClickCount=2 and DoubleClickSpeed=60 achieves double-click instead.

### [SEVERITY: Low] — Hold/Release on mouse correctly excluded for Scroll and Drag
**Scenario:** User somehow has KeyActionType="Hold Down" with Value="Scroll Up".
**Impact:** Code explicitly excludes scroll and drag from hold/release: `if (keyAction == "Hold Down" && !actionStr.StartsWith("Scroll") && !actionStr.Contains("Drag"))`. Correct.
**Verified:** Yes
**Fixed:** N/A

### [SEVERITY: Low] — CoordinateMode reset after MouseClick
**Scenario:** User sets CoordinateMode to "Window" or "Client" for a mouse click.
**Impact:** The compiler emits `CoordMode "Mouse", "{coordMode}"` before the click and `CoordMode "Mouse", "Screen"` after. This correctly sandwiches the click in the right coordinate space without affecting subsequent steps.
**Verified:** Yes
**Fixed:** N/A

---

## Verdict

✓ No actionable bugs found. One dead code path ("Double" click detection) exists but is harmless. The compilation handles all mouse action/target/coordinate combinations correctly.
