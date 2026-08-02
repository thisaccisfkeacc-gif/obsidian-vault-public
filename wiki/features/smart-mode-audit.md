---
tags: [feature, macro-editor, audit]
date: 2026-07-15
sources:
  - PowerX_Keys_V2/ViewModels/MacroEditorViewModel.SmartView.cs
  - PowerX_Keys_V2/ViewModels/MacroEditorViewModel.Core.cs
status: active
---

# Smart Mode Bundling — Edge Case Audit

---

### [SEVERITY: Critical] — Shared modifier steps deleted when one shortcut block removed

**Scenario:** Record Ctrl+C then Ctrl+V (all under one Ctrl hold). Smart Mode shows two blocks: "CTRL + C" and "CTRL + V". Both have the same Ctrl Hold and Ctrl Release objects in their `VirtualSourceSteps`. Delete "CTRL + C". The code protects shared steps by scanning other display blocks — BUT this only works if the display hasn't been refreshed yet. After `RefreshDisplaySteps()`, the "CTRL + V" block is rebuilt fresh with new `VirtualSourceSteps` references from `MapSnapshotToSteps`.

**Impact:** If display refreshes between the protection scan and the actual raw step deletion, the protection set becomes stale. The shared Ctrl Hold/Release could be deleted, breaking Ctrl+V.

**Fix:** Run the protection scan on the raw `MacroSteps` collection using step identity (GUID or object reference), not on the display-view's virtual blocks.

---

### [SEVERITY: Medium] — Text block editing doesn't handle multi-byte/emoji characters

**Scenario:** User records typing an emoji or special character that spans multiple keypresses (e.g., compose sequences on international keyboards). `GetTypingChar` only handles single ASCII chars and "space"/"tab".

**Impact:** The bundled text shows garbled or missing characters. Editing the text block value may produce incorrect raw step reconstruction.

**Fix:** Low priority for now — document as a known limitation. Most users type English/ASCII. If international support is needed later, enhance `GetTypingChar` with a keymap table.

---

### [SEVERITY: Medium] — Scroll bundling absorbs delays without user consent

**Scenario:** User scrolls down 3 times with intentional 400ms pauses between each scroll. Smart Mode bundles them all into "Scroll Down ×3" and absorbs the delay steps into `VirtualSourceSteps`. On playback, the script compiler may not reproduce the original timing.

**Impact:** Fast scroll during playback where user intended slow, deliberate scrolling.

**Fix:** Check how `ScriptCompilerService` handles `ScrollAmount` — if it fires all at once, this is a real bug. If it re-inserts inter-step delays from the source steps, it's fine. Either way, consider a threshold: only bundle if delays between scrolls are < 150ms (very rapid scrolling), not 500ms.

---

### [SEVERITY: Medium] — Double Click bundling proximity too generous (20px)

**Scenario:** User clicks at (100, 100) then clicks at (119, 119) — 20px away. Smart Mode bundles them as "Double Click" even though the user intended two separate clicks at slightly different positions.

**Impact:** Playback does a double-click at the first position instead of two single clicks at different spots.

**Fix:** Reduce proximity threshold from 20px to 5-10px for the "same spot" double-click detection. Or match the first click's threshold (5px) which is already used for the hold/release same-spot check.

---

### [SEVERITY: Medium] — Shortcut block allows non-keyboard steps inside modifier wrap

**Scenario:** User holds Ctrl, moves mouse, presses C, releases Ctrl. The code says `// Non-keyboard/delay — absorb and continue` for non-keyboard steps inside the wrap. This means the mouse move gets absorbed into the shortcut block's `VirtualSourceSteps`.

**Impact:** If the user deletes the "CTRL + C" shortcut block, the mouse move step (which was absorbed) also gets deleted — unexpected loss of a mouse action.

**Fix:** Break out of the wrap scan when a non-keyboard, non-delay step is encountered, rather than absorbing it. Emit whatever keys were collected so far and let the mouse step pass through independently.

---

### [SEVERITY: Medium] — Text bundler stops at Backspace but partially consumes it

**Scenario:** User types "hello" then presses Backspace 3 times (to get "he"), then types "lp" (resulting in "help"). The text bundler handles Backspace by trimming the bundled string, which works. BUT if Backspace is pressed when `bundled` is empty (user starts by deleting), it's skipped with `bundled.Length > 0` check — the Backspace raw step is still consumed into `src` but has no effect.

**Impact:** Deleting the "Type Text" block also deletes those initial Backspace steps, which the user may have wanted to keep as separate key presses.

**Fix:** Minor — only absorb Backspace into the text block if there's already text to delete. If `bundled` is empty when Backspace arrives, break out of the text bundler and let it be emitted as a standalone key press.

---

### [SEVERITY: Low] — CapsLock toggle detection relies on Hold Down only

**Scenario:** The `GetCapsLockStateAt` method flips CapsLock on "Hold Down" events. If the recording misses a CapsLock event or the recorder reports it differently, the caps state could be inverted for the rest of the recording.

**Impact:** Text shows lowercase when it should be uppercase (or vice versa). Cosmetic only — the raw steps still execute correctly.

**Fix:** Accept as cosmetic limitation. The compiled AHK script sends exact key events regardless of Smart View display.

---

### [SEVERITY: Low] — `validWrap` fallback allows shortcut without release

**Scenario:** Recording is interrupted (user stops recording mid-modifier-hold). The code says `if (!validWrap && innerSteps.Any(...key presses...))` — it still emits a shortcut block without the outer modifier Release step.

**Impact:** The virtual block's `VirtualSourceSteps` is incomplete — missing the release. If deleted and re-added in raw mode, the Release step is gone from the macro, leaving a stuck modifier on playback.

**Fix:** When emitting without `outerRelease`, flag the block or add a synthetic release step to `VirtualSourceSteps` so the script compiler always generates a clean Release.

---

### [SEVERITY: Low] — Drag detection threshold differs from click detection

**Scenario:** Hold Down at (100,100), Released Up at (104,104) — 4px apart. `sameSpot` check uses `<= 5`, so this counts as a click. But the actual drag threshold in AHK/Windows is typically 4px. Edge case: user intended a very short drag.

**Impact:** Extremely rare. A 4px drag is indistinguishable from finger movement during a click for most users.

**Fix:** No action needed. Current threshold is reasonable.

---

## Priority Summary

| # | Severity | Issue |
|---|----------|-------|
| 1 | Critical | Shared modifier steps — stale protection after display refresh |
| 2 | Medium   | Text bundler doesn't handle multi-byte/emoji |
| 3 | Medium   | Scroll bundling absorbs intentional delays (500ms threshold too high) |
| 4 | Medium   | Double Click 20px threshold too generous |
| 5 | Medium   | Non-keyboard steps absorbed inside modifier wrap |
| 6 | Medium   | Text bundler consumes empty Backspace |
| 7 | Low      | CapsLock detection cosmetic inaccuracy |
| 8 | Low      | Shortcut without release — incomplete VirtualSourceSteps |
| 9 | Low      | Drag vs click threshold edge case |

## Recommended Fixes (in priority order)

1. **#1 Critical** — Needs fix to prevent data loss on delete
2. **#4 + #3** — Quick threshold adjustments (20→10px, 500→150ms)
3. **#5** — Break wrap scan on non-keyboard steps instead of absorbing

The rest are minor/cosmetic and can wait.

---

## Related Pages

- [[macro-editor]]
- [[undo-redo-audit]]

