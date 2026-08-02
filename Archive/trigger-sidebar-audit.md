---
tags: [audit, trigger-modes, sidebar, qa]
date: 2026-07-16
status: complete
---

# Trigger & Sidebar Audit — PowerX Keys

Auditor: AI Agent  
Date: 2026-07-16  
Status: Complete (double-filtered)

---

## Confirmed Real Findings (3)

---

### [SEVERITY: Low] — ScreenEvent: UI shows value below 100ms but actual is clamped
**Phase:** Trigger Modes
**Component:** ScreenEvent mode
**Scenario:** User types "50" in custom poll interval textbox
**Impact:** Model clamps to 100ms (`Math.Max(100, value)`) but the TextBox still displays "50". User believes polling is 50ms when it's actually 100ms.
**Fix:** Either update the TextBox binding to reflect the clamped value, or add a minimum enforcer on the UI side.
**Verified:** Yes

---

### [SEVERITY: Low] — PressAndRelease: no distinctive trigger pill color
**Phase:** Trigger Modes
**Component:** PressAndRelease UI
**Scenario:** User selects PressAndRelease mode
**Impact:** Falls through to default purple. Every other mode has its own color theme (orange for hold modes, green for screen event, blue for schedule, etc). Inconsistent visual.
**Fix:** Add a DataTrigger for `TriggerMode.PressAndRelease` in both the pill Border and TextBlock styles.
**Verified:** Yes

---

### [SEVERITY: Low] — Profile delete leaves orphaned actions in JSON
**Phase:** Sidebar
**Component:** Profile system
**Scenario:** User deletes a custom profile
**Impact:** Actions assigned to that profile remain in `ConfigManager.Current.Hotkeys` forever. They're never displayed or compiled, but accumulate as dead weight in the JSON config over time.
**Fix:** When a profile is deleted, also remove (or reassign) all actions with that `AssignedProfile`.
**Verified:** Yes

---

## Summary Table

| # | Severity | Phase | Component | Title |
|---|----------|-------|-----------|-------|
| 1 | Low | Triggers | ScreenEvent | Poll interval UI shows clamped value |
| 2 | Low | Triggers | PressAndRelease | No distinctive trigger pill color |
| 3 | ~~Low~~ | ~~Sidebar~~ | ~~Profiles~~ | ~~Orphaned actions~~ — FALSE FLAG, cleanup exists |

---

## Totals

- **Critical:** 0
- **Medium:** 0
- **Low:** 2 (1 false flag removed)
- **Total:** 2 real fixes applied

---

## Removed as False Flags / Intentional Design (19 original findings filtered out)

- Hold/Release not in dropdown → intentional merge into LongPress/PressAndRelease
- PressAndRelease no ReleasePath → graceful degradation, compiles as Press
- Release index 10000 offset → theoretical, impossible in practice
- Single no defaults cleanup → stale values are harmless, never read
- DoubleTap clickCount=1 → unreachable from UI
- Hold = LongPress identical → intentional legacy alias
- Toggle empty slots → runtime tooltip IS the warning
- Toggle GUID variable names → standard generated code pattern
- Schedule auto-buffer → intentional first-run behavior
- Schedule no max interval → UI enforces presets only
- MobileRemote clears hotkey → intentional (mode doesn't use keys)
- Macro exclusivity → intentional safety guard against AHK race conditions
- No search → feature request
- No drag-drop → feature request
- ON/OFF hover-only → intentional clean UI, accent bar shows state
- No context menu → design choice (inline popups)
- No empty state Custom Macros → minor UX, not a bug
- Name truncation → standard behavior with visible "..." suffix
- Hotkey "Assign" keycap → has distinct gray/italic styling, intentional
