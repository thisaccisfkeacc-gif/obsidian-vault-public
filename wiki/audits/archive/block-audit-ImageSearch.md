# Block Audit: ImageSearch (Deep — Cross-Path)

**Date:** 2026-07-24
**Scope:** C# in-process, AHK compiled, AHK preview (SingleStep)

---

## Summary

ImageSearch block finds images on screen using pixel matching. Has Smart Box (pre-cropped region), FindAllMatches (multiple results), and legacy engine fallback.

---

## BUGS

### 1. Smart Box rebasing is preview-only
- **Severity:** Medium
- **Location:** ScriptCompilerService.SingleStep.cs
- **Problem:** Preview applies Smart Box coordinate rebasing (adjusting crop region relative to target window). C# and AHK Full paths do NOT rebase Smart Box coordinates. If a Smart Box was captured on a moved/resized window, preview may find the image but full execution may not.

### 2. FindAllMatches is preview-only
- **Severity:** Low (diagnostic feature)
- **Location:** ScriptCompilerService.SingleStep.cs
- **Problem:** `FindAllMatches` and `MatchSelectMode` only work in single-step test mode. They are not compiled into full AHK scripts. Appears intentional as a diagnostic tool.

### 3. Legacy engine fallback gap
- **Severity:** Low
- **Location:** MacroExecutionService.cs
- **Problem:** When UIA-based image search fails, C# falls back to a legacy GDI+ engine. AHK has no such fallback — it only uses AHK's built-in `ImageSearch`.

---

## DEAD CODE

1. `FindAllMatches` property serialized to JSON but only read in SingleStep preview. Full execution ignores it.
2. `MatchSelectMode` property — same as above.

---

## REDUNDANCIES

1. Scope parsing (WindowName, ClassName, ProcessName) quadruplicated across C#, AHK Full, AHK Preview, and capture callback. Same issue as UIElement block.

---

## MISSING FEATURES

| Feature | C# | AHK Full | AHK Preview |
|---|---|---|---|
| Smart Box rebasing | ❌ | ❌ | ✅ |
| FindAllMatches | N/A | N/A | ✅ |
| Legacy GDI+ fallback | ✅ | ❌ | N/A |
| Tolerance (1-100) | ✅ | ✅ | ✅ |

---

## INCONSISTENCIES

1. Tolerance default — C# defaults to 10, AHK compiled uses `step.Tolerance ?? 10`. Consistent, but the UI slider range (0-100) doesn't match the AHK `ImageSearch` tolerance range (0-255 for color variation). AHK silently clamps.

---

## VERIFIED OK

- Tolerance application — consistent across paths
- Image path resolution — all paths use `%APPDATA%\PowerX_Keys\CaptureImages\{id}.png`
- Coordinate mode (absolute/relative) — handled correctly in all paths
