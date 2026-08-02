# Block Audit: LogicIf (Deep — Cross-Path)

**Date:** 2026-07-24
**Scope:** C# in-process, AHK compiled, AHK preview (SingleStep)

---

## Summary

LogicIf block provides conditional branching — evaluates a condition (ImageFound, PixelFound, WindowExists, UIElementFound, Variable, etc.) and executes true/false child steps.

---

## BUGS

### 1. Broken WPF binding — LogicValue → LogicExpectedValue (FIXED)
- **Severity:** Critical (fixed)
- **Location:** LogicContainerTemplates.xaml:110
- **Problem:** XAML bound `LogicValue` instead of `LogicExpectedValue`. The condition input field wasn't wired to the correct property, so user-entered expected values were lost on save/reload.
- **Fix:** Changed binding to `LogicExpectedValue`.

### 2. Underscore cleaning inconsistency
- **Severity:** Low
- **Location:** ScriptCompilerService.cs vs MacroExecutionService.cs
- **Problem:** AHK strips underscores from variable names (`_myVar` → `myVar`). C# does NOT strip underscores. A variable named `_test` will resolve differently between paths.

---

## DEAD CODE

1. Dead `LogicMode` enums — Some enum values in `LogicConditionType` are defined but never handled in the compiler's switch statement. They fall through to a default "always true" path.

---

## REDUNDANCIES

1. Condition evaluation logic duplicated between C# (`EvaluateCondition`) and AHK (inline conditional expressions). Same logic, different implementations.

---

## MISSING FEATURES

| Feature | C# | AHK Full | AHK Preview |
|---|---|---|---|
| All condition types | ✅ | ✅ | ✅ |
| Nested If/Else | ✅ | ✅ | ✅ |
| Variable comparison | ✅ | ✅ | ✅ |

---

## VERIFIED OK

- True/False child step execution — consistent
- Condition evaluation for ImageFound, PixelFound, WindowExists — consistent
- Token interruption in child steps — consistent
