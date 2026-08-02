---
tags: [feature, reef, powerx-block]
date: 2026-06-13
sources:
  - C:\Users\Maaz\Projects\StayLocked\Reef\src\main\java\dev\pranav\reef\util\AppBlockManager.kt
  - C:\Users\Maaz\Projects\StayLocked\Reef\src\main\java\dev\pranav\reef\accessibility\BlockerService.kt
  - C:\Users\Maaz\Projects\StayLocked\Reef\src\main\java\dev\pranav\reef\screens\AppBlockScreen.kt
  - C:\Users\Maaz\Projects\StayLocked\Reef\src\main\java\dev\pranav\reef\screens\PowerXHomeScreen.kt
status: active
---

# App Blocking (PowerX Block / Reef)

**Summary:** A generic app blocking manager that allows users to restrict access to **any** installed application. Supports two blocking modes: Complete Block and Time Limit with quick presets.

## Blocking Modes

| Mode | Behavior |
|------|----------|
| **Complete Block** | The application is completely blocked and cannot be opened at all. |
| **Time Limit** | The application can be used for a set duration within a specific window before access is blocked. |

## Time Windows (Time Limit mode)

| Window | Reset Logic |
|--------|-------------|
| `/hour` | Resets every clock hour (e.g. 2:00, 3:00) |
| `/6 hours` | Resets every 6 hours from midnight (0:00, 6:00, 12:00, 18:00) |
| `/day` | Resets at midnight |

## Key Files

- [AppBlockManager.kt](file:///C:/Users/Maaz/Projects/StayLocked/Reef/src/main/java/dev/pranav/reef/util/AppBlockManager.kt) — Singleton manager handling data serialization and resetting
- [AppBlockScreen.kt](file:///C:/Users/Maaz/Projects/StayLocked/Reef/src/main/java/dev/pranav/reef/screens/AppBlockScreen.kt) — App list, picker dialog, and preset configuration UI
- [BlockerService.kt](file:///C:/Users/Maaz/Projects/StayLocked/Reef/src/main/java/dev/pranav/reef/accessibility/BlockerService.kt) — Accessibility service enforcing blocks and notifications
- [PowerXHomeScreen.kt](file:///C:/Users/Maaz/Projects/StayLocked/Reef/src/main/java/dev/pranav/reef/screens/PowerXHomeScreen.kt) — Main dashboard navigation card

## Related Pages

- [[overview]]
