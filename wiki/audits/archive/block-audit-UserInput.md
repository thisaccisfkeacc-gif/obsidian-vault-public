# Block Audit: UserInput (Deep — Cross-Path)

**Date:** 2026-07-24
**Scope:** C# in-process, AHK compiled, AHK preview (SingleStep)

---

## BUGS

### 1. AHK YesNo/Dropdown paths don't set LastActionSucceeded
- **Severity:** Medium
- **Location:** ScriptCompilerService.cs:3058-3059
- **Problem:** YesNo MsgBox fallback, custom GUI, and Dropdown paths never set `LastActionSucceeded`. Downstream LogicIf using `AboveStepSuccess`/`AboveStepFailed` reads stale state.

### 2. X-button semantics differ between C# and AHK
- **Severity:** Medium
- **Location:** MacroExecutionService.cs vs ScriptCompilerService.cs
- **Problem:** C# X-button → cancel/abort (macro stops). AHK X-button → continue with default (current selection or "No"). User testing via C# preview gets different results on X-button than compiled AHK.

### 3. No preview support
- **Severity:** Medium
- **Location:** ScriptCompilerService.SingleStep.cs
- **Problem:** UserInput has no branch in SingleStep. Preview generates empty script.

### 4. AvailableVariableNames shows unsanitized names
- **Severity:** Low
- **Location:** MacroEditorViewModel.Properties.cs:1242
- **Problem:** Dropdown shows `"my var"` but runtime stores as `"myvar"`. UI doesn't warn about sanitization.

---

## INCONSISTENCIES

1. Variable name sanitization now consistent (both allow underscore) — but UI dropdown shows unsanitized names.
2. Cancel behavior: C# throws OperationCanceledException. AHK MsgBox YesNo forces a choice (no cancel concept).

---

## VERIFIED OK

- Text InputBox syntax correct
- Dropdown GUI correct
- YesNo GUI correct
- All X-button handlers prevent hang
- Depth limit of 10 consistent across paths
