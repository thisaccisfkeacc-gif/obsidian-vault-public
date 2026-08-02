# Preview Behavior Toggle — Design Decisions

> Discussed: 2026-07-18

## Overview

Add a **single global toggle** in Global Settings to control whether the app minimizes when the user clicks "Preview" on a block.

---

## Global Settings Changes

### 2. Preview Behavior Section (in Global Settings)
Group these toggles under a **"Preview Behavior"** section:

| Toggle | Default | Notes |
|---|---|---|
| Minimize app on Preview | ON | Single toggle covering all applicable blocks |
| Show popup on result | — | |
| Play sound on result | — | |

---

## Block-Level Behavior

### ✅ Controlled by the toggle (minimize is optional)
- Window
- Image
- Pixel
- UI Element
- Show Message / Popup
- Play Sound
- Ask Input
- Launch File

### ❌ Force minimize — mandatory (no toggle)
- Mouse Click
- Type Text
- Keyboard
> These interact with the screen/input directly — app must get out of the way or behavior breaks.

### ❌ Excluded — no preview applies
- Call Macro
> Chains to other macros; preview doesn't make sense here.

### ⚠️ Needs testing
- **Wait for Key** — might need to be force-minimize if the app intercepts the keypress. Test before deciding.

---

## Decision: Single Toggle vs Individual Toggles
- **Decision: Single toggle** — individual toggles per block would clutter the UI.
- One global "Minimize on Preview" toggle covers all applicable blocks listed above.

---

## Implementation Notes
- No changes to per-block UI — toggle lives only in Global Settings.
- Window block: now works without force-minimize since always-on-top is removed. Found window comes to foreground → highlights itself.
- Force-minimize blocks should bypass the toggle entirely in code.
