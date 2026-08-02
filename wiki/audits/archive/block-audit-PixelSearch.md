# Block Audit: PixelSearch (Deep — Cross-Path)

**Date:** 2026-07-24
**Scope:** C# in-process, AHK compiled, AHK preview (SingleStep)

---

## Summary

PixelSearch block finds a specific pixel color on screen. Supports tolerance, scope (screen/window/client), and coordinate modes.

---

## BUGS

### 1. Redundant double PixelSearch call in preview (FIXED)
- **Severity:** Medium (fixed)
- **Location:** ScriptCompilerService.SingleStep.cs
- **Problem:** Preview called `PixelSearch` twice — once for the search, once for verification. This was redundant and could cause timing issues. Fixed to single call.

### 2. Tolerance mismatch between XAML templates
- **Severity:** Low
- **Location:** SearchTemplates.xaml:906 vs :1391
- **Problem:** Two PixelSearch templates define different default tolerance values (Tag=13 vs Tag=10). One template is for the capture flow, the other for manual edit. Users may see different defaults depending on how they created the step.

---

## DEAD CODE

1. `FindAllMatches` on PixelSearch — only used in preview, not compiled to full AHK.

---

## REDUNDANCIES

1. Scope parsing quadruplicated (same as UIElement/ImageSearch).
2. Color conversion (hex string to RGB) duplicated between C# (`ColorTranslator.FromHtml`) and AHK (`Format("0x{:02X}{:02X}{:02X}", r, g, b)`).

---

## MISSING FEATURES

| Feature | C# | AHK Full | AHK Preview |
|---|---|---|---|
| Tolerance (0-100) | ✅ | ✅ | ✅ |
| Scope (screen/window/client) | ✅ | ✅ | ✅ |
| FindAllMatches | N/A | ❌ | ✅ |

---

## VERIFIED OK

- Color matching with tolerance — consistent across paths
- Coordinate mode — handled correctly
- Pixel not found — all paths continue gracefully
