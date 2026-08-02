# Block Audit: SetVariable (Deep — Cross-Path)

**Date:** 2026-07-24
**Scope:** C# in-process, AHK compiled, AHK preview (SingleStep)

---

## BUGS

### 1. AHK value escaping missing backtick and percent
- **Severity:** Medium
- **Location:** ScriptCompilerService.cs:3103-3105
- **Problem:** Only `"` → `""` escaping applied. Backtick (`` ` ``) interpreted as escape sequences. Percent (`%`) triggers AHK variable expansion. A value containing `` `n `` becomes a newline.

### 2. No preview support
- **Severity:** Low
- **Location:** ScriptCompilerService.SingleStep.cs
- **Problem:** SetVariable has no branch in SingleStep. Preview generates empty script.

---

## INCONSISTENCIES

1. AHK SetVariable wraps in try/catch but Map assignment doesn't throw — dead error handling.
2. C# runtime has no try/catch and no success tracking — always returns true.

---

## VERIFIED OK

- Variable name sanitization consistent (alphanumeric + underscore)
- Default variable name `"UserText"` when empty — consistent
- IsValid requires non-empty InputVariableName
