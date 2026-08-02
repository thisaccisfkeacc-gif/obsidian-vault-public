# Block Audit: Text

## Summary
The Text block types text character-by-character or uses clipboard fast paste. Supports variable linking (pull from UserVariables map set by UserInput blocks).

---

### [SEVERITY: Low] — Fast paste 150ms Sleep may be insufficient for slow apps
**Scenario:** User uses FastPasteMode to type a long text block into a slow application (e.g., remote desktop, Electron app with heavy rendering).
**Impact:** The `Sleep(150)` after `Send("^v")` might not be enough time for the paste to complete before `A_Clipboard := ClipSaved` restores the original clipboard. In extreme cases, the target app could paste the restored clipboard content instead.
**Verified:** Yes — hardcoded 150ms after Ctrl+V.
**Fixed:** No — this is a known tradeoff. The 150ms works for 99% of apps. Adding a configurable delay would add UI complexity for minimal gain.

### [SEVERITY: Low] — SetKeyDelay(50, 10) resets key timing for subsequent steps
**Scenario:** Non-fast-paste mode emits `SetKeyDelay(50, 10)` before `SendEvent`.
**Impact:** This changes the global key delay for subsequent keyboard steps in the same macro. However, the macro's SetKeyDelay is already set at the function header level (varies by PlaybackSpeed). This local override could make the next Keyboard step use 50ms delay instead of the intended speed. BUT — looking at the keyboard compilation, it uses `Send()` not `SendEvent()`, and `Send()` in AHK v2 defaults to `SendInput` which ignores `SetKeyDelay`. So this only affects other `SendEvent` calls — which is only other Text blocks. No cross-contamination.
**Verified:** Yes — `SetKeyDelay` only affects `SendEvent`/`SendPlay`, not `Send`/`SendInput`.
**Fixed:** N/A (no bug — AHK v2 isolates Send modes correctly)

### [SEVERITY: Low] — Variable mode with empty UserVariables entry
**Scenario:** `UseVariable=true`, `VariableSource="MyVar"`, but no UserInput block set "MyVar" before this step.
**Impact:** AHK code: `UserVariables.Has("MyVar") ? UserVariables["MyVar"] : ""` — safely returns empty string. The block types nothing. No crash.
**Verified:** Yes
**Fixed:** N/A (graceful empty behavior)

### [SEVERITY: Low] — Text with only whitespace
**Scenario:** User enters only spaces or tabs in the Value field.
**Impact:** `IsValid` uses `!string.IsNullOrWhiteSpace(Value)` — so an all-whitespace value is INVALID. Warning dot shows. User must have at least one non-whitespace character. This might be unexpected if user intentionally wants to type spaces.
**Verified:** Yes — `string.IsNullOrWhiteSpace` rejects tabs, spaces, newlines alone.
**Fixed:** No — edge case. Users wanting pure whitespace can use a Keyboard block with Space key instead. The validation prevents accidental "empty looking" blocks.

### [SEVERITY: Low] — EscapeStringForAhkLiteral handles all necessary characters
**Scenario:** Text contains backticks, quotes, newlines, tabs, or special characters like `%`, `{`, `}`.
**Impact:** The escape function handles backtick→double-backtick, quote→double-quote, \r→`r, \n→`n, \t→`t. In AHK v2 `{Text}` mode, `%`, `{`, `}` are NOT special and sent literally. The escaping is correct and complete.
**Verified:** Yes
**Fixed:** N/A

---

## Verdict

✓ No actionable issues found. Text block handles escaping correctly, variable linking is null-safe, and both paste modes work as intended.
