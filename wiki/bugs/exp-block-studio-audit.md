---
tags: [bug-audit, image-search, experimental]
date: 2026-08-01
status: in-progress
---

# Exp Block — Studio & Cascade Audit

Scan of the Exp block's Image Studio integration and cascade system for potential bugs.

---

## Confirmed Working

- Cascade: Last Position → Smart Box → Window → Full Screen
- Last Position coordinate fix (center → half-width offset)
- Fresh live capture (not frozen screen)
- Source window detection via WindowFromPoint
- FindText code generation from fresh bitmap

---

## Potential Issues Found

### 1. Studio crop doesn't update the saved image file

**Status:** Genuine bug

**What happens:** After cropping in Studio, `step.FindTextCode` is updated (regenerated from cropped bitmap). But `step.SearchImageFilename` still points to the original uncropped PNG file on disk.

**Impact:** If Legacy engine (Turbo OFF) is used, it searches using the image FILE (not FindText code). That file is still the original uncropped image. The search will work but with a larger image than what was shown in Studio.

**Fix:** After Studio saves, overwrite the image file with the cropped bitmap. Or: Studio already overwrites `_originalImagePath` — need to verify.

---

### 2. Studio crop offsets are applied AFTER source window detection

**Status:** Minor / acceptable

**What happens:** Source window position (`SourceWindowX/Y`) is captured before Studio opens. If user crops, `step.X` and `step.Y` get adjusted. But `SourceWindowX/Y` stays the same. This is actually correct — the window position doesn't change just because you cropped the image.

**Impact:** None. Smart Box rebasing still works correctly because the relative position is preserved.

---

### 3. Last Position cache not reset on re-capture

**Status:** Genuine bug

**What happens:** If you capture a NEW image (re-capture), the `LastKnownX/Y/W/H/FindCount` from the previous image remains. The next preview might try Last Position using stale coordinates from the old image.

**Fix:** Clear `LastKnownX = -1`, `LastKnownFindCount = 0` when a new capture completes.

---

### 4. Legacy engine (Turbo OFF) not tested in Exp path

**Status:** Needs verification

**What happens:** The preview compiler (`CompileSingleStepTestScript` — renamed from `CompileExperimentalTestScript`, now in `ScriptCompilerService.SingleStep.cs:9`) has a Legacy ImageSearch cascade path. It uses `ImageSearch(&FoundX, &FoundY, ...)` with the image file path. If the file is the uncropped original (see issue #1), the search area might be too large. Also, Legacy `ImageSearch` returns TOP-LEFT coords, but the C# cache update always stores them as if they're center coords.

**Impact:** After switching from Turbo to Legacy, the Last Position cache coordinates would be wrong (top-left stored as center). The bounding box calculation would be off.

**Fix:** When Legacy is used, don't apply the half-width offset. Or: always store as center by adding half-width to Legacy results before caching.

---

### 5. Find All Matches not implemented in Exp path

**Status:** ~~Not a bug — just not implemented yet~~ **IMPLEMENTED** (verified 2026-08-01)

**What happens:** The `FindAllMatches` property is now checked throughout the Exp preview path (`ScriptCompilerService.SingleStep.cs:437-799`): it skips the Smart Box to search the full configured scope (`_VLeft/_VTop/_VRight/_VBottom`, lines 554-557), and outputs all match coordinates instead of just the first. Full-compile cache bypass also checks it at `ScriptCompilerService.cs:2232`.

**Impact:** "Find All Matches" now highlights all matches in preview.

---

### 6. Studio "Save" overwrites _originalImagePath in-place

**Status:** Needs verification

Looking at line 911: `FinalImagePath = _originalImagePath;`

The Studio saves the cropped bitmap back to the same file path that was passed in. Need to confirm this actually overwrites the file on disk. If it does, issue #1 is not a problem for Legacy engine.

---

## Summary

| # | Issue | Severity | Action |
|---|-------|----------|--------|
| 1 | Cropped image not saved to disk | Not a bug | Studio DOES overwrite file |
| 2 | Crop offsets after window detection | None | Working correctly |
| 3 | Stale Last Position on re-capture | Low | Clear cache on capture |
| 4 | Legacy coords stored as center | Medium | Needs fix if Legacy is used |
| 5 | Find All Matches not implemented | ~~Low~~ | **IMPLEMENTED** (SingleStep.cs:437-799; ScriptCompilerService.cs:2232) |
| 6 | Studio file overwrite behavior | Not a bug | Verified — works |
| 7 | Full compilation (ScriptCompilerService.cs) forces full screen for Exp | By design | Exp override in `CompileSingleStepTestScript` (SingleStep.cs) |
| 8 | Full compilation has no cascade for Exp | By design | Only preview has cascade; full compile just searches full screen |
| 9 | Last Position UI added | Done | Gear menu shows coords + highlight + reset |
