# Block Audit: LoopSequence (Deep — Cross-Path)

**Date:** 2026-07-24
**Scope:** C# in-process, AHK compiled, AHK preview (SingleStep)

---

## BUGS

### 1. Sandbox compiler generates invalid `Loop()` syntax
- **Severity:** High
- **Location:** ScriptCompilerService.cs:4919
- **Problem:** Generates `Loop({loopCount}) {` with parentheses. AHK v2 Loop is a control-flow statement — correct syntax is `Loop {loopCount} {`. Main compiler at line 4071 correctly generates `Loop {loopCount} {`.

### 2. ClickCount=0 defaults to 1 despite "invalid" status
- **Severity:** Low
- **Location:** MacroExecutionService.cs:756, ScriptCompilerService.cs:4070
- **Problem:** `IsValid` returns false for ClickCount=0, but both paths silently default to 1. Step shows red dot but executes fine.

---

## DEAD CODE

1. SingleStep.cs — No handler for `MacroStepType.LoopSequence`. Preview produces empty AHK script.

---

## VERIFIED OK

- Token check inside loop body — consistent
- ChildSteps compiled at correct indent level
- Empty ChildSteps list → no iterations (safe)
- Default ClickCount=1 on creation
