# High-Impact UI Element Targeting & Proximity Research

## Executive Summary
When targeting UI Controls (especially state-changing controls like Toggles, Checkboxes, and Radio Buttons), coordinate distance matching alone can cause target jumping across close elements.

Based on industry research across **UiPath**, **Selenium 4**, **Playwright**, and **Microsoft UIA SDK**, below are the filtered, high-impact strategies for PowerX Keys. Low-impact approaches (hardcoded pixel offsets, deep XPath indexes, brute-force DOM recursion) are excluded.

---

## High-Impact Strategy Breakdown

### 1. Spatial Distance Tolerance + Multi-Point Matching (Implemented)
* Group all candidates within a **30px spatial tolerance buffer** instead of blindly picking the closest center.
* Tie-breakers: AutomationId match > Parent Container match > minimum distance fallback.
* **Status**: Active in `MacroEditorViewModel.Commands.cs:1127` and `MacroExecutionService.cs:2213`.
* **Gap**: `MacroExecutionService.cs:2325` (standard action loop) still uses old code — no tolerance, no tie-breakers. Needs patching.

---

### 2. Parent Container Scoping (Partially Implemented)
* **Why it matters**: 90% of target jumping happens because adjacent controls (like 5 toggles in a row) share identical ClassNames and ControlTypes.
* **How it works**: During capture, record `UIParentAutomationId` or `UIParentClassName`. Use as tie-breaker when multiple candidates fall within tolerance.
* **Status**: Capture service already stores `ParentAutomationId` / `ParentClassName`. Preview and Wait-Until-Gone paths already use it as tie-breaker #2. Standard action loop (`:2325`) does not.
* **Impact**: Eliminates cross-row jumping completely.

---

### 3. Electron/Chromium UIA Tree Unreliability (Root Cause Understanding)
* **Why it matters**: This is the root cause of the AntiGravity toggle bug.
* **How it works**: Chromium only exposes its accessibility tree to UIA when an assistive technology requests it via `WM_GETOBJECT` with `OBJID_CLIENT`. Without this, the tree is incomplete or delayed.
* **When toggles change state in Electron apps**, the bounding box shifts because Chromium recalculates the layout. Native WPF/WinForms apps don't have this problem — their UIA trees are always fully exposed.
* **Implication**: Electron apps will always have less reliable UIA data. Tolerance + parent scoping are essential workarounds until Chromium's UIA provider matures (Chromium 142+ enables `UiaProvider` by default, but adoption is slow).

---

### 4. UIA CacheRequest Mechanism (Future Improvement)
* **What it is**: Microsoft UIA SDK has a built-in caching mechanism (`CacheRequest` / `BuildUpdatedCache`) that lets you snapshot element properties at capture time and compare at execution time.
* **Why it matters**: Instead of re-querying the UIA tree from scratch each time (which gives stale/shifted results in Electron), you can use cached properties to verify you're looking at the same element.
* **Impact**: More robust element identity verification across state changes.
* **Status**: Not implemented. Candidate for v3.

---

### 5. Relative Text Anchor Locators (Future - UiPath Style)
* Link a target control to the nearest static text label captured during selection (e.g., "Enable Dark Mode" label next to a toggle).
* During execution, find the anchor text first, then locate the target relative to it.
* **Impact**: 100% resilient to layout refactoring. Best for controls without unique AutomationId.
* **Status**: Candidate for v3.

---

## Excluded Low-Impact Approaches (Noise)
* Hardcoded Pixel Offsets — fragile across resolution changes.
* Deep Index Paths — breaks on any UI update.
* Full Screen Image Fallback — heavy CPU/RAM; UIA pattern verification is superior.
* Async Invoke Pattern Guard — valid but separate concern (prevents hangs, not target jumping).

---

## Known Gaps
1. **Third code path unfixed**: `MacroExecutionService.cs:2325` (standard action finding loop) still uses simple proximity without tolerance, tie-breakers, or parent scoping.
2. **Parent scoping not in standard action loop**: Even after tolerance is added to `:2325`, parent container tie-breaker needs to be added there too.

---

## Roadmap
1. **Now (Done)**: Spatial Tolerance Buffer (30px) + AutomationId Tie-Breaker.
2. **Now (In Progress)**: Apply same fix to third code path (`:2325`).
3. **Next**: Parent Container Scoping as primary filter (not just tie-breaker) — scope search to parent before spatial matching.
4. **Future (v3)**: UIA CacheRequest for element identity verification + Relative Text Anchors.
