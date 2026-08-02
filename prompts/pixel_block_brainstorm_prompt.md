# 🧠 Brainstorming Request: Pixel Block Architecture & Improvements

We are seeking creative ideas to improve the **Pixel Block (PixelSearch step)** in PowerX Keys — a Windows desktop macro automation app built with C# .NET 10, WPF, AutoHotkey v2, and SQLite.

---

## 🎯 What Is The Pixel Block?

The **Pixel Block** is a macro step that:
1. At **recording time**: the user picks a single target pixel color from the screen (via a color picker / eyedropper tool). The exact hex color (e.g. `#2D9CDB`) and the screen coordinates (`X`, `Y`) where it was sampled are stored.
2. At **runtime**: searches a screen region for any pixel that matches that stored color (within a configurable tolerance). When found, outputs the matched coordinates (`FoundX`, `FoundY`) which subsequent steps (like Click, Move Mouse, Wait Until, etc.) can consume.

Unlike the **Image Block** (which matches a bitmap template), the Pixel Block only matches a **single color value**. It is much faster but far less specific — it finds the *first pixel* of that color anywhere in the search region, not a particular shape or icon.

---

## ⚙️ How The Pixel Block Works (AHK v2 Compiled Output)

The block compiles into an AHK v2 script via `ScriptCompilerService.cs`. Here is a simplified version of what gets generated:

### Primary Search (Tier 1)
```ahk
LastActionSucceeded := 0
if PixelSearch(&FoundX, &FoundY, x1, y1, x2, y2, 0x2D9CDB, 10) {
    LastActionSucceeded := 1
    ; optional OffsetX/OffsetY adjustment
}
```

### Self-Healing Fallback (Tier 2 — always active)
If Tier 1 fails, automatically expands the search area by ±2px in every direction and retries:
```ahk
if (LastActionSucceeded == 0) {
    if PixelSearch(&FoundX, &FoundY, Max(0, x1-2), Max(0, y1-2), x2+2, y2+2, 0x2D9CDB, 10) {
        LastActionSucceeded := 1
    }
}
```
This silently recovers from sub-pixel rendering shifts (e.g. 1px anti-aliasing drift, minor DPI rounding errors).

### Failure Handling
```ahk
if (LastActionSucceeded == 0) {
    ToolTip("🛑 Pixel not found")
    SetTimer(() => ToolTip(), -2000)
    return ; Abort — no fallback handler
}
```

---

## 🔧 Key MacroStep Properties for Pixel Block

| Property | Purpose |
|---|---|
| `TargetColorHex` | The 6-char hex color string to search for (e.g. `2D9CDB`) |
| `Tolerance` | Color matching tolerance (0–255). 0 = exact match. Default ~10. |
| `X`, `Y` | Recorded coordinates where the color was originally sampled |
| `SearchWidth`, `SearchHeight` | Dimensions of the recorded search region |
| `SmartSearchBoxSize` | Smart Box padding around the recorded position (default 60px) |
| `SearchScopeSummary` | Scope: `Full Screen`, `SMART_BOX`, `WIN_LIVE:...`, `WIN_REL:...`, `WIN_SMART:...` |
| `EnableSmartRetry` | Retry loop on failure (up to `MaxRetries` times with `RetryDelayMs` delay) |
| `FailIfMissing` | Abort macro if pixel not found |
| `OffsetX`, `OffsetY` | Fine-tune output coordinates offset from found pixel |
| `StepName` | Named variable prefix for multi-step access (e.g. `ButtonPixel` → `ButtonPixelX`, `ButtonPixelY`) |
| `IsDebugHighlight` | Flash a magenta ripple animation at the found pixel location for debugging |

---

## 🗺️ Scope System (Same as Image Block)

The Pixel Block shares the same scope parsing system as Image Block:

| Scope | Behavior |
|---|---|
| `Full Screen` | Searches entire screen (0, 0, A_ScreenWidth, A_ScreenHeight) |
| `SMART_BOX` | Searches a tight box around the originally recorded position + padding |
| `WIN_LIVE:appname` | Dynamically resolves the current bounds of a running app window at runtime |
| `WIN_REL:appname` | Same but uses coordinates relative to the window's top-left corner |
| `WIN_SMART:appname` | WIN_REL with automatic window fallback always enabled |
| Fixed coords | Hardcoded pixel rectangle |

---

## 🐞 Current Pain Points & Known Limitations

1. **Color-Only Matching is Extremely Fragile**: A single pixel of the right color anywhere in the search region triggers a match — even if it belongs to a completely different UI element that happens to share the same color.
2. **No Shape Context**: Unlike ImageSearch, PixelSearch has zero awareness of what surrounds the found pixel. It cannot distinguish "blue button background" from "blue border" from "blue icon".
3. **Theme / Dark Mode Sensitivity**: If the app changes its color theme, the sampled hex color no longer exists anywhere and the search always fails.
4. **Anti-Aliasing & Sub-Pixel Rendering**: Semi-transparent pixels near edges may not exactly match the sampled color even with tolerance, causing false negatives on specific monitors or DPI settings.
5. **Tolerance is Coarse**: A single tolerance value (0–255) applies uniformly to all color channels (R, G, B). There's no way to say "allow large variation in brightness but keep the hue close".
6. **No Fallback Chain Beyond +2px**: The self-healing fallback only tries a 2px expansion. If the color drifted further (e.g. window moved 10px), the block fails completely.
7. **No Window Fallback Tier for PixelSearch**: Unlike Image Block which has a 3-tier cascade (Smart Box → Window → Full Screen), Pixel Block only has 2 tiers (Search Region → Search Region +2px). There's no Window-level fallback.
8. **Color Picker UX is Basic**: The user picks one static hex color. There's no way to pick a color range, a gradient zone, or multiple alternative colors to try.
9. **No Multi-Color Support**: Cannot search for "any of these 3 colors" in one block. Users must chain multiple Pixel Blocks to cover UI state variants.
10. **PixelSearch Scan Order**: AHK's `PixelSearch` scans left-to-right, top-to-bottom. If the target pixel is in the bottom-right corner of a full-screen search, it's slow. There's no way to hint the search to start from the last known position.

---

## 💡 Seed Ideas (Not Finalized)

- **Color Range Picker**: Instead of one hex value, let the user define a hue range (e.g. "any blue between #2A8FD0 and #3AAFFF"). More resilient to slight shade changes.
- **Multi-Color OR Logic**: "Find a pixel that is red OR orange OR yellow" — covers hover/active/normal states of the same UI element.
- **Anchor Pixel Pattern**: Instead of one pixel, record a small 3×3 or 5×5 grid of pixel colors. All must match within tolerance. Much more specific than a single pixel — eliminates false positives.
- **Nearest Named Color**: At capture time, identify the nearest CSS/design named color (e.g. "Sky Blue", "Coral") so users understand what they captured and can re-capture intelligently if the theme changes.
- **Window Fallback Tier**: Add a 3rd tier matching Image Block — if Smart Box fails, try the full window bounds before giving up.

---

## 🚀 Your Instructions

Please evaluate the current Pixel Block system and provide:
1. **Assessment of current architecture**: What is solid vs. brittle in the current 2-tier system?
2. **Ideas to make color matching more reliable**: How do we reduce false positives and false negatives without making it as heavy as Image Block?
3. **Ideas to improve the recording/capture UX**: How should users pick their target color?
4. **Any radical rethinks**: Is there a fundamentally better approach than single-pixel color matching for PowerX Keys' use case that remains lightweight?

Write your feedback and ideas directly into this file under your agent section!

---

## ✏️ Agent Responses

### Agent 1 Feedback & Ideas:

**1. Architecture Assessment**

Solid: The code is clean and correct. The +2px self-healing fallback is a smart lightweight fix for sub-pixel drift. The scope system (SMART_BOX, WIN_LIVE, etc.) is shared cleanly with Image Block.

Brittle: 2 tiers are not enough. Image Block has 3 tiers, Window fallback, and cache. Pixel Block has nothing beyond the basic search. Users expect the same robustness but PixelSearch fails the moment anything shifts by more than 2px.

**2. Ideas for More Reliable Matching**

- **Per-Channel Tolerance**: Instead of one flat tolerance (0–255) applied to all RGB channels, allow separate R, G, B tolerances. For example, "tolerance 20 on R, 10 on G, 50 on B" lets you be generous with blue brightness while keeping red tight. This catches theme changes that shift brightness but preserve hue.

- **Anchor Pattern (3×3 pixel grid)**: At capture time, record a 3×3 grid of pixel colors around the sampled point. At search time, require all 9 pixels to match (within their individual tolerances). This eliminates false positives — a single random pixel of the right color won't trigger a match. The search is still fast because it's just 9 `PixelGetColor` calls per candidate position using a `Loop` over the scan region.

- **Multi-Color Fallback List**: Let the user record 2–3 alternative hex colors at capture time. The search tries each color in sequence. If `#2D9CDB` fails, try `#3AAFFF`. If that fails, try `#1A7BBB`. This handles dark-mode/light-mode or hover/normal state changes without needing the heavier Image Block.

**3. Capture UX Improvements**

- **Live Color Picker with Range**: When the user clicks to sample, show a small magnifier that also highlights all pixels within the current tolerance on screen in real-time, so the user can immediately see how specific (or noisy) their match is before closing the picker.

- **Nearest Named Color**: Show a label like "This color is closest to **Sky Blue (#2D9CDB)**" during capture. This helps users understand what they're matching and re-capture intelligently if the app theme changes.

**4. Radical Rethink**

Replace single-pixel `PixelSearch` with a **2-pass approach**:
- Pass 1: `PixelSearch` for the target color (fast, same as today)
- Pass 2 (optional): If found, verify the match by checking a 5×5 pixel neighborhood around the found position. If fewer than 50% of the surrounding pixels match within tolerance, it's likely a false positive. Reject and continue scanning.

This adds ~1ms per candidate and eliminates ~90% of false positives while keeping the speed of single-pixel search.

---

### Agent 2 Feedback & Ideas:
*(Write your thoughts and suggestions here)*

---

### Agent 3 Feedback & Ideas:
*(Write your thoughts and suggestions here)*
You can paste this directly under **Agent 2**.

---

## **1. Architecture Assessment**

The current Pixel Block is intentionally lightweight, which is its biggest strength. However, it relies on **one piece of evidence (a single color)** to make a decision. In computer vision terms, that's extremely weak evidence.

Instead of making Pixel Block behave more like Image Block, I would strengthen the evidence while keeping it lightweight.

Think of Pixel Block as evolving from:

> "Find this pixel."

into

> "Verify this visual signature."

---

## **2. Novel Idea: Pixel Signature**

Instead of recording only one pixel, automatically record a tiny signature around it.

Example:

```text
       Green
Blue   Target   Gray
Black  White    Blue
```

Store:

* Center color
* 4 cardinal neighbors
* 4 diagonal neighbors

Only 9 pixels.

Runtime:

1. Find candidate center pixel.
2. Instantly verify surrounding colors.

This eliminates most false positives while remaining dramatically faster than ImageSearch.

---

## **3. Relative Color Signature**

Instead of storing absolute RGB values,

store relationships.

Example:

```text
Center

Top is darker

Left is brighter

Bottom is blue

Right is gray
```

Even if Dark Mode changes brightness,

the relationships often remain identical.

Much more theme resistant.

---

## **4. Smart Tolerance**

Current tolerance treats RGB equally.

Instead allow:

```text
Hue

Saturation

Brightness
```

Example:

```text
Hue:
±3°

Brightness:
±40

Saturation:
±20
```

Most UI themes change brightness far more than hue.

Searching in HSV/HSL space would greatly reduce failures.

---

## **5. Pixel Confidence Score**

Instead of

Found / Not Found

calculate

```text
95%

83%

61%
```

Confidence could consider:

* Color distance
* Neighbor verification
* Signature match
* Search tier

Then expose:

Minimum Confidence

Example:

```text
Accept only

>85%
```

---

## **6. Adaptive Search Expansion**

Current system:

```text
+2 px
```

Instead:

```text
+2

↓

+6

↓

+15

↓

Smart Box

↓

Window

↓

Full Screen
```

Very similar philosophy to Image Block.

Small UI shifts stay extremely fast.

Large shifts still recover.

---

## **7. Last Successful Direction**

Instead of remembering only

Last X/Y

also remember movement.

Example:

```text
Usually drifts

+18 px

-2 px
```

Next search begins there instead of the old location.

Over time Pixel Block "learns" where the UI usually moves.

---

## **8. Capture Assistant**

After selecting a pixel,

PowerX Keys analyzes it.

Example:

```
This color appears 642 times
inside the selected region.

⚠ Weak choice.
```

or

```
Unique color detected.

Excellent anchor.
```

Users immediately know whether they picked a reliable pixel.

---

## **9. Auto-Promote to Better Block**

Sometimes Pixel Block isn't the right tool.

Example:

User clicks

```
OK
```

The recorder notices

* text
* icon
* button border

Suggestion:

> This target may be more reliable as an Image Block or OCR Block.

The software helps beginners choose the correct automation strategy.

---

## **10. Pixel Cluster Mode**

Instead of finding

one pixel,

find

multiple nearby pixels.

Example:

```
Need

5 matching pixels

within

12×12 px
```

This avoids matching random isolated pixels while remaining much lighter than ImageSearch.

---

## **11. Live Reliability Meter**

While hovering with the eyedropper,

display:

```
Reliability

★★★★★

Unique

Fast

Stable
```

or

```
★☆☆☆☆

Avoid

Common color

Appears 18,000 times
```

Users naturally capture better anchors.

---

## **12. Pixel History**

Every successful execution updates statistics.

```
Found

842 times

Average

6 ms

Failures

3

Average drift

9 px
```

If reliability starts dropping,

PowerX Keys warns:

> This Pixel Block has become unreliable. Consider recapturing.

---

## **13. Radical Rethink: Context-Aware Pixel Block**

Rather than being "Pixel Search",

make it a lightweight **Visual Anchor Block**.

Internally it decides what the minimum evidence should be.

For example:

* Single unique pixel → classic PixelSearch
* Small color signature → Pixel Signature
* Gradient → Gradient Signature
* Text edge → OCR hint
* Icon corner → Image Block suggestion

The user still thinks they're creating a "Pixel Block," but PowerX Keys automatically chooses the most reliable lightweight detection strategy based on what was captured. This preserves Pixel Block's speed while dramatically improving robustness without turning it into a full image-matching engine.

### Agent 2 Feedback & Ideas:

**1. Architecture Assessment**

**Solid**
- The +2px self-healing fallback is an elegant, zero-cost fix for the most common real-world failure mode (sub-pixel anti-aliasing and DPI rounding).
- Sharing the exact same scope system (`SMART_BOX`, `WIN_LIVE`, `WIN_REL`, `WIN_SMART`) with Image Block is the right design decision — one consistent mental model for users.
- The compiled output is lean and predictable. PixelSearch remains one of the fastest primitives available in AHK.

**Brittle**
- Only two tiers is a clear regression versus Image Block’s three-tier cascade. Once the color moves more than 2px (window resize, DPI change, layout shift), the block has nowhere to go.
- No Window-level fallback means `WIN_SMART` scopes are less useful than they should be.
- Pure single-pixel matching has no concept of “this is the right blue” versus “any blue pixel”. False positives are inevitable on colorful UIs.
- Failure handling is abrupt (tooltip + abort). There is no soft recovery path or diagnostic information about *why* the color disappeared.

**2. Ideas to Make Color Matching More Reliable**

- **Hue-Preserving Tolerance Mode**  
  Add a second tolerance mode: “Hue + Saturation locked, Lightness flexible”. Convert both the target and candidate pixels to HSL (or HSV). Allow a larger delta on the Lightness channel while keeping Hue within a tight band (e.g. ±8°). This survives most theme changes and brightness shifts without needing multiple colors.

- **Mini-Cluster Match (not full 3×3 grid)**  
  At capture time, automatically sample the center pixel + the four cardinal neighbors (N/S/E/W). At runtime require the center + at least 3 of the 4 neighbors to match within tolerance. This is dramatically more specific than a single pixel yet still extremely cheap (5 `PixelGetColor` calls). False positives from random matching pixels almost disappear.

- **Last-Known-Position Bias + Expanding Search**  
  Always start the search from the last successful coordinates (or the recorded X/Y). First try a tiny 8×8 box, then expand by 50% each miss (8 → 12 → 18 → 27…) up to the Smart Box limit, then fall through to Window / Full Screen. This turns the common case (“pixel barely moved”) into an almost instant hit while still providing graceful degradation.

- **Color Variant Set (OR logic)**  
  Allow the user to store 1–4 alternative hex values for the same logical pixel (normal / hover / pressed / dark-mode). The search tries them in priority order at every tier. One block covers the entire visual state family.

- **Confidence Score via Neighborhood Density**  
  After a candidate is found, quickly count how many of the surrounding 8 pixels are within tolerance. Expose this as a 0–100 “density” score. Users can set a minimum density threshold (e.g. “at least 5 of 8 neighbors must also match”). Rejects sparse false positives while remaining lightweight.

**3. Capture / Recording UX Improvements**

- **Live Magnifier + Tolerance Heatmap**  
  When the eyedropper is active, show a 12× zoom magnifier that also paints every pixel currently matching the chosen color + tolerance in a translucent overlay. The user instantly sees whether their selection is unique or will match half the screen.

- **One-Click “Capture Cluster”**  
  Beside the normal eyedropper, offer a “Capture Cluster” button. It records the center color plus the four cardinal neighbors automatically and stores them as a Mini-Cluster. Zero extra work for the user, much higher specificity.

- **Theme Snapshot Pair**  
  After the first capture, offer “Also capture Dark/Light variant?”. If accepted, the system temporarily forces the system theme (or asks the user to toggle) and samples the same relative location again. Both colors are stored as a Variant Set in one step.

- **Named Color + Delta Preview**  
  Display both the exact hex and the nearest named color (“Closest to #2D9CDB — Dodger Blue”). Also show a small delta readout: “Current screen pixel is Δ12 from target”. Helps users understand sensitivity before they close the picker.

- **Scope Override Chip**  
  Immediately after capture, show a small chip: “Detected as SMART_BOX on explorer.exe” with a one-click “Force Full Screen” or “Force Window” override. Prevents the Progman/WorkerW pollution problem at the source.

**4. Radical Rethinks (still lightweight)**

- **Hybrid Pixel + Relative Anchor**  
  Keep pure PixelSearch as the primary engine, but optionally let the user also pick a second “anchor” pixel a short distance away (e.g. 20px left). At runtime the search first finds the primary color, then verifies that the relative anchor color also exists at the expected offset. Extremely cheap extra check that adds shape-like context without turning into ImageSearch.

- **Directional Search Hint**  
  Allow the user to mark the expected direction the pixel usually appears relative to the last known position (Above / Below / Left / Right / Center). The search starts from that side of the Smart Box and expands outward. On full-screen searches this can cut average search time dramatically for bottom-right targets.

- **Soft Pixel Block → Image Block Promotion**  
  If a Pixel Block fails repeatedly in the same macro run, the runtime can silently offer (or auto-promote) to a tiny Image Block by capturing a 16×16 region around the last known good location. Gives users a graceful upgrade path when pure color matching proves insufficient.

**Immediate Highest-ROI Fixes**
1. Add a proper Window Fallback tier (make Pixel Block’s cascade match Image Block).  
2. Implement Last-Known-Position biased expanding search.  
3. Add Mini-Cluster (center + 4 neighbors) as an optional matching mode.  
4. Live tolerance heatmap in the color picker.

These four changes keep the Pixel Block lightning-fast while closing the biggest reliability and UX gaps that currently force users to switch to the heavier Image Block.


You can paste this directly under **Agent 2**.

---

## **1. Architecture Assessment**

The current Pixel Block is intentionally lightweight, which is its biggest strength. However, it relies on **one piece of evidence (a single color)** to make a decision. In computer vision terms, that's extremely weak evidence.

Instead of making Pixel Block behave more like Image Block, I would strengthen the evidence while keeping it lightweight.

Think of Pixel Block as evolving from:

> "Find this pixel."

into

> "Verify this visual signature."

---

## **2. Novel Idea: Pixel Signature**

Instead of recording only one pixel, automatically record a tiny signature around it.

Example:

```text
       Green
Blue   Target   Gray
Black  White    Blue
```

Store:

* Center color
* 4 cardinal neighbors
* 4 diagonal neighbors

Only 9 pixels.

Runtime:

1. Find candidate center pixel.
2. Instantly verify surrounding colors.

This eliminates most false positives while remaining dramatically faster than ImageSearch.

---

## **3. Relative Color Signature**

Instead of storing absolute RGB values,

store relationships.

Example:

```text
Center

Top is darker

Left is brighter

Bottom is blue

Right is gray
```

Even if Dark Mode changes brightness,

the relationships often remain identical.

Much more theme resistant.

---

## **4. Smart Tolerance**

Current tolerance treats RGB equally.

Instead allow:

```text
Hue

Saturation

Brightness
```

Example:

```text
Hue:
±3°

Brightness:
±40

Saturation:
±20
```

Most UI themes change brightness far more than hue.

Searching in HSV/HSL space would greatly reduce failures.

---

## **5. Pixel Confidence Score**

Instead of

Found / Not Found

calculate

```text
95%

83%

61%
```

Confidence could consider:

* Color distance
* Neighbor verification
* Signature match
* Search tier

Then expose:

Minimum Confidence

Example:

```text
Accept only

>85%
```

---

## **6. Adaptive Search Expansion**

Current system:

```text
+2 px
```

Instead:

```text
+2

↓

+6

↓

+15

↓

Smart Box

↓

Window

↓

Full Screen
```

Very similar philosophy to Image Block.

Small UI shifts stay extremely fast.

Large shifts still recover.

---

## **7. Last Successful Direction**

Instead of remembering only

Last X/Y

also remember movement.

Example:

```text
Usually drifts

+18 px

-2 px
```

Next search begins there instead of the old location.

Over time Pixel Block "learns" where the UI usually moves.

---

## **8. Capture Assistant**

After selecting a pixel,

PowerX Keys analyzes it.

Example:

```
This color appears 642 times
inside the selected region.

⚠ Weak choice.
```

or

```
Unique color detected.

Excellent anchor.
```

Users immediately know whether they picked a reliable pixel.

---

## **9. Auto-Promote to Better Block**

Sometimes Pixel Block isn't the right tool.

Example:

User clicks

```
OK
```

The recorder notices

* text
* icon
* button border

Suggestion:

> This target may be more reliable as an Image Block or OCR Block.

The software helps beginners choose the correct automation strategy.

---

## **10. Pixel Cluster Mode**

Instead of finding

one pixel,

find

multiple nearby pixels.

Example:

```
Need

5 matching pixels

within

12×12 px
```

This avoids matching random isolated pixels while remaining much lighter than ImageSearch.

---

## **11. Live Reliability Meter**

While hovering with the eyedropper,

display:

```
Reliability

★★★★★

Unique

Fast

Stable
```

or

```
★☆☆☆☆

Avoid

Common color

Appears 18,000 times
```

Users naturally capture better anchors.

---

## **12. Pixel History**

Every successful execution updates statistics.

```
Found

842 times

Average

6 ms

Failures

3

Average drift

9 px
```

If reliability starts dropping,

PowerX Keys warns:

> This Pixel Block has become unreliable. Consider recapturing.

---

## **13. Radical Rethink: Context-Aware Pixel Block**

Rather than being "Pixel Search",

make it a lightweight **Visual Anchor Block**.

Internally it decides what the minimum evidence should be.

For example:

* Single unique pixel → classic PixelSearch
* Small color signature → Pixel Signature
* Gradient → Gradient Signature
* Text edge → OCR hint
* Icon corner → Image Block suggestion

The user still thinks they're creating a "Pixel Block," but PowerX Keys automatically chooses the most reliable lightweight detection strategy based on what was captured. This preserves Pixel Block's speed while dramatically improving robustness without turning it into a full image-matching engine.


### Agent 2 Feedback & Ideas:

**1. Architecture Assessment**

**Solid**
- The +2px self-healing fallback is an elegant, zero-cost fix for the most common real-world failure mode (sub-pixel anti-aliasing and DPI rounding).
- Sharing the exact same scope system (`SMART_BOX`, `WIN_LIVE`, `WIN_REL`, `WIN_SMART`) with Image Block is the right design decision — one consistent mental model for users.
- The compiled output is lean and predictable. PixelSearch remains one of the fastest primitives available in AHK.

**Brittle**
- Only two tiers is a clear regression versus Image Block’s three-tier cascade. Once the color moves more than 2px (window resize, DPI change, layout shift), the block has nowhere to go.
- No Window-level fallback means `WIN_SMART` scopes are less useful than they should be.
- Pure single-pixel matching has no concept of “this is the right blue” versus “any blue pixel”. False positives are inevitable on colorful UIs.
- Failure handling is abrupt (tooltip + abort). There is no soft recovery path or diagnostic information about *why* the color disappeared.

**2. Ideas to Make Color Matching More Reliable**

- **Hue-Preserving Tolerance Mode**  
  Add a second tolerance mode: “Hue + Saturation locked, Lightness flexible”. Convert both the target and candidate pixels to HSL (or HSV). Allow a larger delta on the Lightness channel while keeping Hue within a tight band (e.g. ±8°). This survives most theme changes and brightness shifts without needing multiple colors.

- **Mini-Cluster Match (not full 3×3 grid)**  
  At capture time, automatically sample the center pixel + the four cardinal neighbors (N/S/E/W). At runtime require the center + at least 3 of the 4 neighbors to match within tolerance. This is dramatically more specific than a single pixel yet still extremely cheap (5 `PixelGetColor` calls). False positives from random matching pixels almost disappear.

- **Last-Known-Position Bias + Expanding Search**  
  Always start the search from the last successful coordinates (or the recorded X/Y). First try a tiny 8×8 box, then expand by 50% each miss (8 → 12 → 18 → 27…) up to the Smart Box limit, then fall through to Window / Full Screen. This turns the common case (“pixel barely moved”) into an almost instant hit while still providing graceful degradation.

- **Color Variant Set (OR logic)**  
  Allow the user to store 1–4 alternative hex values for the same logical pixel (normal / hover / pressed / dark-mode). The search tries them in priority order at every tier. One block covers the entire visual state family.

- **Confidence Score via Neighborhood Density**  
  After a candidate is found, quickly count how many of the surrounding 8 pixels are within tolerance. Expose this as a 0–100 “density” score. Users can set a minimum density threshold (e.g. “at least 5 of 8 neighbors must also match”). Rejects sparse false positives while remaining lightweight.

**3. Capture / Recording UX Improvements**

- **Live Magnifier + Tolerance Heatmap**  
  When the eyedropper is active, show a 12× zoom magnifier that also paints every pixel currently matching the chosen color + tolerance in a translucent overlay. The user instantly sees whether their selection is unique or will match half the screen.

- **One-Click “Capture Cluster”**  
  Beside the normal eyedropper, offer a “Capture Cluster” button. It records the center color plus the four cardinal neighbors automatically and stores them as a Mini-Cluster. Zero extra work for the user, much higher specificity.

- **Theme Snapshot Pair**  
  After the first capture, offer “Also capture Dark/Light variant?”. If accepted, the system temporarily forces the system theme (or asks the user to toggle) and samples the same relative location again. Both colors are stored as a Variant Set in one step.

- **Named Color + Delta Preview**  
  Display both the exact hex and the nearest named color (“Closest to #2D9CDB — Dodger Blue”). Also show a small delta readout: “Current screen pixel is Δ12 from target”. Helps users understand sensitivity before they close the picker.

- **Scope Override Chip**  
  Immediately after capture, show a small chip: “Detected as SMART_BOX on explorer.exe” with a one-click “Force Full Screen” or “Force Window” override. Prevents the Progman/WorkerW pollution problem at the source.

**4. Radical Rethinks (still lightweight)**

- **Hybrid Pixel + Relative Anchor**  
  Keep pure PixelSearch as the primary engine, but optionally let the user also pick a second “anchor” pixel a short distance away (e.g. 20px left). At runtime the search first finds the primary color, then verifies that the relative anchor color also exists at the expected offset. Extremely cheap extra check that adds shape-like context without turning into ImageSearch.

- **Directional Search Hint**  
  Allow the user to mark the expected direction the pixel usually appears relative to the last known position (Above / Below / Left / Right / Center). The search starts from that side of the Smart Box and expands outward. On full-screen searches this can cut average search time dramatically for bottom-right targets.

- **Soft Pixel Block → Image Block Promotion**  
  If a Pixel Block fails repeatedly in the same macro run, the runtime can silently offer (or auto-promote) to a tiny Image Block by capturing a 16×16 region around the last known good location. Gives users a graceful upgrade path when pure color matching proves insufficient.

**Immediate Highest-ROI Fixes**
1. Add a proper Window Fallback tier (make Pixel Block’s cascade match Image Block).  
2. Implement Last-Known-Position biased expanding search.  
3. Add Mini-Cluster (center + 4 neighbors) as an optional matching mode.  
4. Live tolerance heatmap in the color picker.

These four changes keep the Pixel Block lightning-fast while closing the biggest reliability and UX gaps that currently force users to switch to the heavier Image Block.