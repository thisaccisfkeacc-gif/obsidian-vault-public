# Block Audit: Keyboard

## Summary
The Keyboard block handles single keys, combos (Ctrl+C), and Hold/Release states. Compilation uses `{Blind}` mode for single keys and modifier prefix notation for combos.

---

### [SEVERITY: Medium] — `{Blind}` prefix may interfere with modifier-aware apps
**Scenario:** A single key is compiled as `Send("{Blind}" . "{Enter}")`. The `{Blind}` prefix means AHK won't release any physically held modifiers before sending the key.
**Impact:** If the user is physically holding Shift when the macro runs, `{Blind}` causes the sent key to also include Shift (e.g., Shift+Enter instead of Enter). This is intentional for recording fidelity (it preserves the state at recording time), but could surprise users who manually add keyboard steps.
**Verified:** Yes — line: `sb.AppendLine($"{indent}Send(\"{{Blind}}\" . \"{outKey}\")")`
**Fixed:** No — this is a design choice, not a bug. The `{Blind}` mode is correct for recorded macros. A toggle could be added later but isn't a defect.

### [SEVERITY: Low] — Value="None" treated as invalid but not cleared on key capture cancel
**Scenario:** User starts key capture, then cancels. Value stays as whatever it was before (or "None" if freshly added from the palette).
**Impact:** `IsValid` correctly rejects "None" as invalid (`!string.Equals(Value, "None", ...)`). Warning dot shows. No functional bug — just noting the "None" string is used as a sentinel value rather than null.
**Verified:** Yes
**Fixed:** N/A (correct behavior)

### [SEVERITY: Low] — Combo with single "+" key: SplitCombo correctly identifies literal plus
**Scenario:** User records just the "+" key (OemPlus). The value could be `+` or `{+}`.
**Impact:** `SplitCombo` looks for `+` separators. If value is just `+`, the last character IS `+`, so `lastIsKey = true` (empty string after last `+`). Then `separatorCount = plusIndices.Count - 1 = 0`, so it returns null (not a combo). Falls through to single-key path. The single-key path checks `"+^!#{}".Contains(outKey[0])` and wraps it in braces: `{+}`. Correct behavior.
**Verified:** Yes
**Fixed:** N/A (not a bug)

### [SEVERITY: Low] — Hold Down on a combo (e.g., "Ctrl + A" with Hold)
**Scenario:** User has a combo like "lcontrol + a" and sets KeyActionType to "Hold Down".
**Impact:** `ConvertComboHoldReleaseToAhk` generates `{Ctrl Down}{a Down}`. This is correct AHK v2 syntax for holding both keys down simultaneously. The reverse (Release) generates `{a Up}{Ctrl Up}` (reversed order). Correct.
**Verified:** Yes
**Fixed:** N/A (correct)

### [SEVERITY: Low] — IsKeyCombo always returns false
**Scenario:** The `IsKeyCombo` property is hardcoded to `false` with comment "Dropdown always visible when value exists (XAML hides when empty)".
**Impact:** The XAML uses `IsKeyCombo` to collapse the tri-state Press/Hold/Release button for combos. Since it's always false, the tri-state button is ALWAYS visible — even for combos. However, the compiler still respects `KeyActionType` for combos via `ConvertComboHoldReleaseToAhk`, so the user CAN set Hold/Release on combos and it compiles correctly.
**Verified:** Yes
**Fixed:** N/A (intentional — the old logic that hid it was removed to give users combo hold/release)

---

## Verdict

✓ No actionable issues found. The Keyboard block compilation is robust with good handling of combos, special keys, and modifier characters.
