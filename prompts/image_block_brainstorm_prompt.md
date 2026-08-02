# 🧠 Brainstorming Request: Image Block Architecture & Cascade System Improvements

We are seeking creative, high-performance, and flaw-free ideas to improve the **Image Block (ImageSearch step)** in PowerX Keys — a Windows desktop macro automation app built with C# .NET 10, WPF, AutoHotkey v2, and SQLite.

---

## 🎯 What Is The Image Block?

The **Image Block** is a macro step that:
1. Takes a **screenshot of a UI region or image** at recording time and saves it as a `.png` file.
2. At runtime, searches the screen (or a scoped region) for that saved image using AHK v2's `ImageSearch()` function.
3. When found, outputs the **center coordinates** (`FoundX`, `FoundY`, `FoundW`, `FoundH`) which subsequent steps (like Click, Move Mouse, etc.) can consume.

There are **two engines**:
- **Standard Engine**: Uses AHK v2 `ImageSearch()` — pixel-by-pixel bitmap comparison with a tolerance slider (0–255).
- **Fast Engine**: Uses a third-party `FindText()` AHK library — converts the image to a base64-encoded text pattern at capture time, enabling fast cross-DPI text/shape matching without storing a file.

---

## ⚙️ How The Cascade System Works (3-Tier Fallback)

The Image Block compiles into an AHK v2 script with up to **3 fallback tiers** via `ScriptCompilerService.cs`:

### Tier 1 — Primary Search (Always runs)
Search within the configured scope:
- `Full Screen` → `ImageSearch(0, 0, A_ScreenWidth, A_ScreenHeight, ...)`
- `Smart Search / SMART_BOX` → A tight bounding box around the original recorded position + configurable padding (default 60px). This is the fastest option.
- `WIN_LIVE:` / `WIN_REL:` / `WIN_SMART:` → Dynamically resolves the live window bounds of a specific app at runtime using `WinGetPos()`, then searches inside that window only.
- `Fixed Coords` → Fixed pixel rectangle.

### Tier 2 — Window Fallback (Optional, `hasWindowFallback`)
If Tier 1 fails AND the scope was window-relative (`WIN_LIVE`, `WIN_REL`, `WIN_SMART`):
```ahk
} else if (_WinHwnd && ImageSearch(&FoundX, &FoundY, _WX, _WY, _WX+_WW, _WY+_WH, image)) {
```
Searches the **entire captured window** as a wider safety net.

### Tier 3 — Full-Screen Fallback (Optional, `hasSmartSearchFallback`)
If the scope was `Smart Search` and Tier 1 still failed (e.g. desktop icon moved to another monitor):
```ahk
} else if (ImageSearch(&FoundX, &FoundY, _VLeft, _VTop, _VRight, _VBottom, image)) {
```
Searches the **entire virtual screen** (all monitors combined) as a last resort.

---

## 🔧 Key MacroStep Properties for Image Block

| Property | Purpose |
|---|---|
| `SearchImageFilename` | Path to the `.png` template image |
| `FindTextCode` | Base64 fast-engine pattern (alternative to PNG) |
| `ImageTolerance` | Pixel color tolerance (0–255, default ~10) |
| `SearchScopeSummary` | Scope string: `Full Screen`, `SMART_BOX`, `WIN_LIVE:...`, `WIN_REL:...`, `WIN_SMART:...` |
| `X`, `Y` | Recorded center coordinates of the image |
| `SearchWidth`, `SearchHeight` | Recorded image dimensions |
| `SmartSearchBoxSize` | Smart box padding (default 60px) |
| `EnableSmartRetry` | Retry loop on failure (up to `MaxRetries` times with `RetryDelayMs` delay) |
| `FailIfMissing` | Abort macro if image not found after all tiers |
| `OffsetX`, `OffsetY` | Fine-tune click target offset from image center |
| `UseFastEngine` | Use FindText instead of AHK ImageSearch |
| `IsDebugHighlight` | Flash a red border around matched region for 2s |

---

## 🐞 Current Pain Points & Known Limitations

0. **⚠️ Preview vs Master Compiler Mismatch (Critical)**: The 3-tier cascade (Smart Box → Window → Full Screen) exists ONLY in the preview compiler (`ScriptCompilerService.SingleStep.cs`). The master compiler (`ScriptCompilerService.cs`) has only 2 tiers and is missing the Full-Screen fallback entirely. Desktop captures fail silently during real macro execution while preview shows success.
0. **Desktop Capture Scope Pollution**: When drag-capturing an image on the desktop, `WindowFromPoint` returns the `Progman`/`WorkerW` window (owned by `explorer.exe`). This incorrectly sets `SearchImageSourceAppName = "explorer"` and `SearchScopeSummary = "WIN_SMART:explorer:..."` instead of `"Smart Search"`. This causes `hasWindowFallback = true` and the tooltip says "Window Fallback" even though it's a desktop search.
0. **Last Known Position Cache (Step 0)**: The preview script has a Step 0 that checks a cached bounding box around the last found position. If the image hasn't moved far, it skips the full 3-tier cascade. The master executor does NOT use this cache.
0. **FindAllMatches Mode**: The block supports finding ALL matches (not just first) with configurable `MatchSelectMode` (First/Last). Useful for finding multiple UI elements of the same type.
0. **Scope Detection During Capture**: The capture overlay uses `WindowFromPoint` + class filtering (`Progman`/`WorkerW`) to determine scope. If a window is detected → `WIN_SMART:processName:coords`. If no window (desktop) → `"Smart Search"`. This logic has gaps (see pain point 0 above).
1. **DPI Scaling Issues**: On high-DPI monitors (150%, 200%), captured images are scaled differently at runtime. AHK `ImageSearch` compares raw pixels, so a 96dpi-captured image may not match on a 144dpi screen.
2. **Theme/Color Changes**: If an app changes its color theme or uses OS dark/light mode, the pixel colors change and the image no longer matches even at high tolerance.
3. **Animated / Dynamic UI**: Buttons with hover states, loading spinners, animated icons will not match the static template.
4. **Smart Box Too Tight**: If the UI shifts (app window moved slightly), Tier 1 fails, requiring Tier 2/3, which are much slower (full screen scan).
5. **No Confidence Score**: `ImageSearch` is binary — found or not found. There's no way to know "how confident" the match is or pick the best match from multiple candidates.
6. **FindText (Fast Engine) Capture is Fragile**: The base64 pattern must be re-captured if the font, icon, or rendering changes even slightly.
7. **Image Studio UX**: Users must manually open the Image Studio window to crop/edit images. There's no live preview of what will be matched at runtime.
8. **No Multi-Image Fallback**: If the primary image isn't found, there's no way to try an alternative image template (e.g. try dark-mode image, then light-mode image).

---

## 💡 Potential Areas for Improvement (Seed Ideas — Not Finalized)

- **Template Variants**: Allow the user to attach 2–3 alternative template images to a single Image Block (e.g. dark mode + light mode versions). The cascade tries each variant before escalating to the next tier.
- **Perceptual Hashing / Feature Matching**: Use OpenCV or Windows ML to do perceptual image comparison instead of raw pixel comparison — DPI and minor color differences become irrelevant.
- **Live Match Preview**: Show a small floating preview window during recording that overlays the matched region in real-time so the user can confirm the image will be detected correctly.
- **Confidence Threshold**: Instead of binary found/not-found, expose a 0–100% match confidence slider and allow the user to tune sensitivity.
- **Smart Box Auto-Expand**: If Tier 1 fails, automatically double the Smart Box size and retry before escalating to full-screen Tier 3.

---

## 🚀 Your Instructions

Please evaluate the current system and provide:
1. **Assessment of the current cascade system**: What is architecturally strong vs. brittle?
2. **Your own novel ideas** for making Image Block more reliable, especially under DPI changes, theme changes, and dynamic UI.
3. **Ideas for improving Image Block UX** in the macro editor (recording flow, feedback, configuration).
4. **Any radical rethinks**: Is there a fundamentally better approach than pixel-comparison `ImageSearch` for PowerX Keys' use case?

Write your feedback and ideas directly into this file under your agent section!

---

## ✏️ Agent Responses

### Agent 1 Feedback & Ideas:

**1. Cascade Assessment**

Strong: The 3-tier fallback is architecturally solid — narrowest scope first (fast path), expand on failure. The Smart Box with configurable padding is the right default. The `_WinHwnd` guard on Window Fallback correctly skips it when no source window exists.

Brittle: The preview compiler and master compiler have diverged. The master compiler is missing Tier 3 (Full Screen fallback) entirely, which means desktop captures silently fail at runtime. This needs to be the first fix — backport the full 3-tier cascade from `SingleStep.cs` to `ScriptCompilerService.cs`.

**2. Novel Ideas**

- **Multi-Variant Template Cascade**: Before Tier 1, try multiple image variants (e.g. `image_dark.png`, `image_light.png`, `image_highdpi.png`) in sequence. Only escalate to Tier 2/3 if ALL variants fail at the current tier level. This handles theme/DPI changes without expanding the search area.

- **Adaptive Smart Box**: If Tier 1 fails AND the image was previously found at a Last Known Position, don't jump to Tier 2 immediately. First expand the Smart Box by 2x and retry Tier 1. Only fall through to Tier 2/3 if the expanded box also fails. This handles small UI shifts gracefully.

- **Floating Preview Window During Capture**: After capturing, show a small live overlay that runs `ImageSearch` in real-time over the captured region so the user can immediately verify the match works before closing the capture dialog.

**3. UX Improvements**

- Show the fallback tier that succeeded in the macro editor UI (e.g. "Found via Full Screen Fallback") so the user knows their Smart Box is too tight.
- Add a "Test Match" button in the Image Block settings that runs a one-shot preview and reports which tier hit + match time.

**4. Radical Rethink**

Consider replacing `ImageSearch` with **OpenCV template matching** (`MatchTemplate` + `MinMaxLoc`) via a small native helper. Benefits:
- Sub-pixel matching with confidence scores (0–1.0)
- DPI-invariant when using normalized correlation methods
- Multi-match result sorting by confidence
- Masked matching (ignore transparent/background areas in the template)

This is a big lift but would solve DPI, theme, and confidence-score problems in one shot. For now, harmonizing the preview and master compilers is the highest-ROI fix.

---

### Agent 2 Feedback & Ideas:
*(Write your thoughts and suggestions here)*

---

### Agent 3 Feedback & Ideas:
*(Write your thoughts and suggestions here)*
You can paste this directly under **Agent 2**.

---

## **1. Cascade Assessment**

The current cascade (Smart Box → Window → Full Screen) is a good foundation because it optimizes for the common case first. However, every retry currently expands the **search area** while still relying on the **same matching strategy**. In other words, only *where* you search changes—not *how* you search.

I would separate the cascade into two independent dimensions:

* **Search Scope Cascade** (Smart Box → Window → Desktop → Full Screen)
* **Matching Strategy Cascade** (Exact → High Tolerance → Variant Template → Fast Engine → Alternative Engine)

This allows PowerX Keys to recover from different failure types more intelligently instead of always scanning a larger area.

---

## **2. Novel Idea: Detection Pipeline Instead of Simple Cascade**

Instead of treating ImageSearch as one operation, treat every Image Block as a detection pipeline.

Example:

```
Stage 1
---------
Smart Box
Exact Template
Tolerance 5

↓

Stage 2
---------
Smart Box
Tolerance 20

↓

Stage 3
---------
Smart Box
Dark Theme Variant

↓

Stage 4
---------
Window Search
Primary Template

↓

Stage 5
---------
Window Search
Alternative Template

↓

Stage 6
---------
Full Screen
```

Most failures are actually caused by rendering differences—not by the image moving somewhere else.

Searching smarter is often faster than searching bigger.

---

## **3. Capture "Environment Fingerprint"**

When recording an image, also store metadata:

* DPI scale
* Monitor resolution
* Windows Light/Dark Mode
* Accent color
* Process name
* Window title
* Window class
* Theme (if detectable)

At runtime:

If these values changed, PowerX Keys immediately knows why matching might fail.

The editor could even display:

> "This image was recorded on 125% DPI. Current monitor is 150%."

or

> "Recorded in Light Mode. Current app appears to be Dark Mode."

This dramatically improves debugging.

---

## **4. Image Health Score**

Each Image Block receives a health score.

Example:

```
★★★★★
Very Reliable

or

★★☆☆☆
Fragile Capture
```

Score factors:

* Image size
* Edge detail
* Contrast
* Number of unique colors
* Presence of text
* Transparency
* Flat-color percentage

Tiny gray icons would receive a warning before the user even saves them.

Example:

> "This image contains only 3 unique colors and may produce unreliable matches."

This prevents bad captures instead of fixing them later.

---

## **5. Automatic Image Quality Analyzer**

Immediately after capture, run analysis.

Detect:

* Too blurry
* Too small
* Too transparent
* Low contrast
* Mostly empty pixels
* Animated content
* Gradient-heavy regions

Suggest improvements automatically.

Example:

> "Capture a slightly larger area including the button border."

or

> "Avoid capturing the animated spinner."

---

## **6. Smart Capture Suggestions**

If the user captures:

```
[ OK ]
```

Automatically suggest

> "Would you like to include the button border for better detection?"

or

> "Detected text-only capture. Consider using OCR Block instead."

The recorder becomes an assistant instead of just a screenshot tool.

---

## **7. Self-Learning Search Region**

Every successful match updates statistics.

Instead of remembering only the last position, maintain:

```
Found 142 times

Average X Drift:
+14 px

Average Y Drift:
-3 px

Maximum Drift:
41 px
```

The Smart Box can automatically resize based on real usage instead of a fixed 60px padding.

Reliable images stay extremely fast.

Moving UI automatically gets a larger search area.

---

## **8. Detection Report**

Add a Debug Report.

```
Image Block #12

✓ Smart Box
18 ms

✗ Variant 2

✓ Window Search
34 ms

Confidence:
Good

Matched:
Attempt 2

Retry:
0
```

Users instantly understand why something was slow.

---

## **9. Image Studio "Stress Test"**

Instead of testing one screenshot,

simulate different environments.

Example:

```
125% DPI

150% DPI

Dark Mode

Light Mode

Brightness

Gamma

Blur

JPEG Compression
```

Estimate whether the image will still match.

Think of it as unit testing for an image template.

---

## **10. Image Collections**

Instead of one PNG, allow an Image Block to own a collection.

Example:

```
Chrome

├── Light.png
├── Dark.png
├── Hover.png
├── Disabled.png
├── High DPI.png
```

PowerX Keys manages them automatically.

The user still edits only one Image Block.

---

## **11. Image Reliability Dashboard**

Across the entire macro:

```
Macro Reliability

Image Blocks

🟢 18 Excellent

🟡 5 Moderate

🔴 2 Fragile
```

Clicking a fragile block explains why.

This becomes incredibly valuable in large automation projects.

---

## **12. Radical Rethink: Multi-Strategy Detection Block**

Rather than an "Image Block," evolve it into a generic **Visual Detection Block**.

Internally it can choose the best strategy:

* ImageSearch
* FindText
* OCR
* OpenCV Template Matching
* Feature Matching (ORB/SIFT/AKAZE)
* Windows UIAutomation (when applicable)

The user simply specifies:

> "Find this visual element."

PowerX Keys automatically selects the most reliable detection engine based on what was captured.

This future-proofs the architecture without forcing users to understand the underlying technology.

### Agent 2 Feedback & Ideas:

**1. Cascade Assessment**

**Strong**
- Narrow-to-wide progression (Smart Box → Window → Full Screen) is the correct performance hierarchy.
- Window-relative scopes (`WIN_LIVE` / `WIN_REL` / `WIN_SMART`) with a true window-bounds fallback is excellent for multi-monitor and window-move scenarios.
- The existence of a Last-Known-Position cache (even if only in preview) shows good instinct for temporal locality.

**Brittle / Critical**
- The preview/master compiler divergence is a production landmine. Desktop captures that succeed in the step editor will silently fail (or take the slow path) at runtime. This must be fixed first — the master compiler must receive the exact same 3-tier logic (including the virtual-screen Tier 3).
- Desktop pollution (`Progman`/`WorkerW` → `WIN_SMART:explorer`) is a silent correctness bug. It forces an unnecessary Window Fallback tier and pollutes tooltips/UI.
- No shared “search strategy” object means the cascade logic is duplicated and will drift again the next time someone touches only one compiler.
- Binary success/failure with no confidence or “why it failed” telemetry makes debugging painful for users.

**2. Novel Reliability Ideas**

- **Scoped Cache + Predictive Expand**  
  Keep a tiny per-Image-Block cache of the last successful search rectangle + the tier that found it. On the next run, start with that exact rectangle (even tighter than Smart Box). If it misses, expand by 1.5× once, then fall into the normal cascade. This turns “small UI shifts” into near-instant hits without ever touching full-screen.

- **Dual-Engine Parallel Race (Standard + Fast)**  
  When both a PNG and a FindText pattern exist, fire both searches concurrently (AHK can do this with a short timeout). Take the first positive hit. This gives the speed of FindText when it works and the robustness of pixel ImageSearch when the pattern is stale.

- **Theme-Aware Variant Sets with Priority**  
  Allow 1–3 named variants per Image Block (Light / Dark / High-Contrast / Custom). At runtime the cascade tries the “preferred” variant first (user-selected or auto-detected via a quick system theme check), then the others at the same tier before expanding the search area. No need to escalate scope just because the theme changed.

- **Soft-Match with Confidence Gate**  
  After a successful `ImageSearch`, immediately sample a small set of key pixels (or run a fast secondary correlation) and compute a simple 0–100 confidence. If confidence is below a user-tunable threshold, treat it as a miss and continue the cascade. This filters out weak matches caused by partial occlusion or minor theme shifts without requiring OpenCV.

- **Dynamic Smart-Box Inflation with Decay**  
  If Tier 1 fails, temporarily inflate the Smart Box by a factor that grows with consecutive failures (1.5× → 2× → 3×) but decays back to normal after a successful find. Prevents permanent “fat” search regions while still handling temporary layout shifts.

**3. UX Improvements for the Macro Editor**

- **Capture-Time Live Validation Overlay**  
  The moment the user finishes the drag-capture, a semi-transparent floating panel appears for 3–4 seconds showing:  
  - the exact region that will be searched (Smart Box outline)  
  - a real-time “Match / No Match” badge using the current engine + tolerance  
  - which scope was auto-detected (and a one-click override button: “Force Smart Search” / “Force Window”)  
  This kills the “I captured it but it never finds it” class of support tickets.

- **Tier Success Badge in the Step List**  
  After a test run or real execution, show a small colored chip on the Image Block:  
  `Smart` (green) / `Window` (yellow) / `Full` (orange) / `Fail` (red).  
  Hovering reveals the exact search rectangle and match time. Users instantly see when their Smart Box is too tight.

- **One-Click “Make Robust” Wizard**  
  A button that:  
  1. Offers to capture a second variant (dark/light).  
  2. Suggests increasing Smart Box padding based on recent failure history.  
  3. Turns on Adaptive Expand + Soft-Match confidence.  
  Turns advanced reliability features into a guided 15-second flow.

- **Image Studio Inline Preview**  
  Replace the separate Image Studio window with an expandable panel inside the Image Block properties. Show the template, a live screen thumbnail of the current Smart Box, and a “Test Now” button that runs the full cascade and highlights the match (or shows why it failed).

**4. Radical Rethinks**

- **Hybrid Visual + Structural Locator**  
  Keep ImageSearch as the primary engine, but optionally attach a lightweight UIAutomation “anchor” (e.g. the nearest button, text label, or control with a stable AutomationId/Name). At runtime:  
  1. Resolve the UIA anchor.  
  2. Search for the image only inside a small relative offset box from that anchor.  
  This survives theme changes, DPI changes, and moderate layout shifts far better than pure pixels, while still allowing pure-image captures when no good anchor exists.

- **Perceptual Hash Fast-Path**  
  At capture time compute a perceptual hash (pHash or dHash) of the template. At runtime, first do a very cheap full-screen (or Smart Box) hash scan of candidate regions. Only run the expensive pixel `ImageSearch` on the top 3–5 hash candidates. Gives near-FindText speed with far better resilience to minor visual changes.

- **Long-term: Replace pure pixel matching with a small ONNX model**  
  Train (or use a pre-trained) lightweight template-matching model that outputs bounding box + confidence. Ship it as a single ~5–10 MB ONNX file and call it from a native helper. Completely eliminates DPI, theme, and anti-aliasing issues. This is the eventual destination if PowerX Keys wants image search to feel “magical” rather than “pixel-perfect or fail”.

**Highest-ROI immediate actions**
1. Unify the 3-tier cascade into both compilers.  
2. Fix desktop (`Progman`/`WorkerW`) classification so it never becomes a window scope.  
3. Port the Last-Known-Position cache to the master executor.  
4. Add the live validation overlay at capture time.

These four changes alone will eliminate the majority of current “it works in the editor but not when I run the macro” reports.