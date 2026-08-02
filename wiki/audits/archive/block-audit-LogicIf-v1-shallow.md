# Block Audit: LogicIf

## Summary
LogicIf provides conditional branching with 6 condition modes: AboveStepSuccess, AboveStepFailed, NamedBlockSuccess, NamedBlockFailed, VariableEquals, VariableNotEquals. Supports True/False branches with nested child steps.

---

### [SEVERITY: Low] — LastActionSucceeded initialized to 1 (AboveStepSuccess is true by default)
**Scenario:** LogicIf with "AboveStepSuccess" mode is the first step in a macro (no prior action).
**Impact:** `global LastActionSucceeded := 1` in executor header means the first LogicIf evaluating "above step succeeded" will be TRUE. This is intentional — "no failure has occurred yet" is equivalent to "success state". If user wants "only if something specific succeeded", they should use NamedBlock mode.
**Verified:** Yes
**Fixed:** N/A (correct initialization semantics)

### [SEVERITY: Low] — VariableEquals/NotEquals treats unset variable as empty string
**Scenario:** LogicIf checks `VariableEquals` for a variable that was never set by a UserInput block.
**Impact:** Code: `(UserVariables.Has("X") ? UserVariables["X"] : "") == "Y"`. Unset variable = empty string. So `VariableEquals` with expected="" would be TRUE for unset variables. This matches the C# preview behavior documented in the code comment.
**Verified:** Yes
**Fixed:** N/A (documented behavior)

### [SEVERITY: Low] — IsSourceDisabled skips entire LogicIf including all children
**Scenario:** User disables the source step that a LogicIf references (e.g., the ImageSearch block it depends on is disabled).
**Impact:** `IsSourceDisabled=true` → compiler emits a comment and returns immediately. ChildSteps and ChildStepsFalse are never compiled. This is correct — if the source step is disabled, the condition can never be meaningful.
**Verified:** Yes
**Fixed:** N/A

### [SEVERITY: Low] — NamedBlock mode sanitizes LogicSource to alphanumeric
**Scenario:** LogicSource references a step named "Find ✅ Button" (with emoji and spaces).
**Impact:** `string cleanTarget = new string(step.LogicSource.Where(c => char.IsLetterOrDigit(c)).ToArray())` → "FindButton". This must match the sanitized StepName used in `StepSuccessStates[...]` by the referenced step. Since all steps use the same sanitization, names match correctly.
**Verified:** Yes
**Fixed:** N/A

### [SEVERITY: Low] — Nesting depth unlimited
**Scenario:** User nests LogicIf blocks 10+ levels deep.
**Impact:** Indent increases by 4 spaces per level. No hard limit on nesting. AHK v2 handles deep nesting fine. The only practical concern is readability of generated code, but that's a non-issue for machine-generated scripts.
**Verified:** Yes
**Fixed:** N/A

### [SEVERITY: Low] — Else branch only compiled if ChildStepsFalse has items
**Scenario:** LogicIf with only a True branch (no Else).
**Impact:** `if (step.ChildStepsFalse != null && step.ChildStepsFalse.Count > 0)` — only emits `} else {` block if there are actual false-branch steps. A LogicIf without an Else branch compiles to a simple `if (...) { ... }`. Correct.
**Verified:** Yes
**Fixed:** N/A

### [SEVERITY: Low] — LogicElse and LogicEndIf enum values exist but aren't compiled
**Scenario:** Old V1-imported macros might have flat LogicElse/LogicEndIf step entries.
**Impact:** `IsValid` returns `true` for LogicElse/LogicEndIf (they're "just structural"). The compiler doesn't have explicit handling for these types — they'd fall through without generating any AHK code. In V2, the nesting model (ChildSteps/ChildStepsFalse) replaces flat Else/EndIf markers. Legacy entries are harmless no-ops.
**Verified:** Yes
**Fixed:** N/A (backward compat, no harm)

---

## Verdict

✓ No actionable issues found. LogicIf compilation correctly handles all 6 condition modes with proper AHK v2 syntax, null-safe variable checks, and clean nesting with Else support.
