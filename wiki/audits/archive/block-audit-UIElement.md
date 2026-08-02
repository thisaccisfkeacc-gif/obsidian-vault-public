# Block Audit: UIElement (Deep — Cross-Path)

**Date:** 2026-07-24
**Scope:** C# in-process, AHK compiled, AHK preview (SingleStep)

---

## Summary

UIElement block finds on-screen elements by UIA properties (Name, AutomationId, Class, ControlType) with 3-tier proximity matching. Previously had a critical spatial proximity bug where preview matched differently from execution.

---

## BUGS

### 1. [FIXED] Spatial proximity matching — preview matched closest element, C# used tolerance + AutomationId + Parent
- **Severity:** Critical (fixed)
- **Location:** MacroEditorViewModel.Commands.cs (preview), MacroExecutionService.cs (C#), ScriptCompilerService.cs/SingleStep.cs (AHK)
- **Problem:** Preview picked the absolute closest element. C# execution used a 3-tier fallback (tolerance → AutomationId → Parent Container). AHK used yet another approach.
- **Fix:** All 3 locations now use the same 3-tier proximity matching logic.

---

## DEAD CODE

1. `AbsoluteX`/`AbsoluteY` properties on MacroStep — `[JsonIgnore]`, set during recording, never consumed by any execution path. Display-only.
2. ParentContainer fallback in AHK compiled path — AHK doesn't have UIA parent traversal, so ParentContainer matching is C#/preview only.

---

## REDUNDANCIES

1. Scope parsing (WindowName, ClassName, ProcessName) is quadruplicated across C# execution, AHK compiled, AHK preview, and capture callback. Should be a shared utility.
2. UIA property extraction repeated in MacroExecutionService, ScriptCompilerService, and UIElementCaptureService with different property dictionaries.

---

## MISSING FEATURES

| Feature | C# | AHK Full | AHK Preview |
|---|---|---|---|
| 3-tier proximity | ✅ | ✅ | ✅ (after fix) |
| Parent Container fallback | ✅ | ❌ (no UIA parent) | ✅ |
| UIA property filtering | ✅ | ❌ (uses Name only) | Partial |
| Smart Box scope parsing | ✅ | ✅ | ✅ |

---

## INCONSISTENCIES

1. AHK compiled uses `UIA` library for element finding. C# uses `System.Windows.Automation`. Different UIA implementations — AHK's UIA library may find elements C# doesn't and vice versa.
2. AHK compiled only matches by Name property. C# matches by Name, AutomationId, Class, ControlType with fallbacks.

---

## VERIFIED OK

- 3-tier proximity matching — consistent across all 3 paths (after fix)
- Scope rebasing — preview correctly rebases relative coordinates to screen
- Element not found — all paths continue gracefully (no crash)
