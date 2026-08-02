# Block Audit: CallMacro (Deep — Cross-Path)

**Date:** 2026-07-24
**Scope:** C# in-process, AHK compiled, AHK preview (SingleStep)

---

## BUGS

### 1. macroCallStack ID-vs-Name mismatch allows first-level self-recursion
- **Severity:** Medium
- **Location:** ScriptCompilerService.cs:1002-1005 vs :4372
- **Problem:** Initial stack entry pushes `targetMacro.Id.ToString()` (numeric ID), but recursion check compares by `calledMacro.Name` (string name). First level of self-recursion silently passes through before Name entries can catch it.

### 2. AHK inline expansion doesn't scope variables
- **Severity:** Low
- **Location:** ScriptCompilerService.cs:4390-4391
- **Problem:** Steps inlined directly at call site with no scoping. If called macro defines local variables, they collide with caller's scope.

### 3. No preview support
- **Severity:** Low
- **Location:** ScriptCompilerService.SingleStep.cs
- **Problem:** CallMacro has no branch in SingleStep. Preview generates empty script.

---

## INCONSISTENCIES

1. Error messages: C# shows WPF MessageBox (blocking). AHK shows ToolTip (non-blocking, disappears after 3s).
2. Sandbox CallMacro has no recursion error message or depth limit UI — silently skips.
3. Recursion warning uses `safeMacroName` (escaped). "Not found" uses `step.Value.Replace()` (different escaping).

---

## VERIFIED OK

- Depth limit of 10 consistent across all 3 paths
- Case-insensitive macro name comparison
- macroCallStack push/pop balanced
- Steps correctly inlined via CompileStepCollection
