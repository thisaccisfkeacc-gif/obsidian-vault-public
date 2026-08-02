# Block Audit: WaitUntil

## Summary
WaitUntil pauses execution until a screen condition is met (ImageFound, PixelFound, WindowActive, WindowExists) or timeout occurs. Shares scope parsing with ImageSearch/PixelSearch. Uses While loop to preserve outer A_Index.

---

### [SEVERITY: Low] — Uses While loop instead of Loop (preserves parent A_Index)
**Scenario:** WaitUntil inside a LoopSequence block.
**Impact:** `While (_ConditionMet == 0) {` instead of `Loop {` — this preserves the parent loop's `A_Index` variable. If `Loop` were used, the inner loop would shadow the parent's counter. Smart design.
**Verified:** Yes — comment in code: "We use a While loop instead of a Loop to preserve the outer A_Index"
**Fixed:** N/A

### [SEVERITY: Low] — Timeout calculated from UITimeoutSeconds or Duration
**Scenario:** User sets timeout to 10 seconds.
**Impact:** `int timeoutMs = step.UITimeoutSeconds > 0 ? step.UITimeoutSeconds * 1000 : (step.Duration ?? 10000)`. Falls back to Duration property, then 10 seconds default. Correct priority chain.
**Verified:** Yes
**Fixed:** N/A

### [SEVERITY: Low] — OnTimeoutAction "Stop" aborts, "Continue" proceeds
**Scenario:** Condition not met within timeout.
**Impact:** If OnTimeoutAction == "Stop": emits `return` (abort macro). Otherwise: sets `LastActionSucceeded := 0` and continues. LogicIf blocks after can check the failure state.
**Verified:** Yes
**Fixed:** N/A

### [SEVERITY: Low] — FoundX/FoundY only updated for Image/Pixel conditions
**Scenario:** WaitUntil with WindowActive condition succeeds.
**Impact:** Code comment: "Only update FoundX/FoundY for conditions that actually capture coordinates. WindowActive/WindowExists don't produce a pixel location." Preserves prior search coordinates correctly.
**Verified:** Yes
**Fixed:** N/A

### [SEVERITY: Low] — Token check inside While loop
**Scenario:** Kill switch pressed during WaitUntil.
**Impact:** `if (myToken != GlobalMacroToken) return` — checked every iteration (every `CheckIntervalMs` milliseconds, default 250ms). Max response time: 250ms.
**Verified:** Yes
**Fixed:** N/A

### [SEVERITY: Low] — Scope parsing identical to ImageSearch
**Scenario:** WaitUntil with WIN_LIVE or Smart Search scope.
**Impact:** Uses the exact same scope parsing code pattern (WIN_LIVE, WIN_REL, WIN_SMART, Smart Box, Full Screen, raw coords). Consistent behavior with ImageSearch.
**Verified:** Yes
**Fixed:** N/A

---

## Verdict

✓ No actionable issues found. WaitUntil is cleanly implemented with proper timeout handling, condition checking, and scope parsing.
