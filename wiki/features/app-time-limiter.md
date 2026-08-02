---
tags: [feature, reef, powerx-block]
date: 2026-06-13
sources:
  - C:\Users\Maaz\Projects\StayLocked\Reef\src\main\java\dev\pranav\reef\util\AppTimeLimiter.kt
  - C:\Users\Maaz\Projects\StayLocked\Reef\src\main\java\dev\pranav\reef\accessibility\BlockerService.kt
  - C:\Users\Maaz\Projects\StayLocked\Reef\src\main\java\dev\pranav\reef\screens\PowerXHomeScreen.kt
status: active
---

# App Time Limiter (PowerX Block / Reef)

**Summary:** A flexible per-app usage time limiter for Instagram and YouTube. Users set daily/hourly limits via quick-tap presets, and the app kicks them to the home screen when the limit is exceeded.

## How It Works

1. User picks a **time** (10m, 15m, 30m, 1h) and a **window** (/hour, /6h, /day)
2. When a tracked app is in the foreground, usage is accumulated
3. When the limit is exceeded → **kicked to home screen** + notification
4. Timer resets at clock-aligned boundaries (e.g. :00 for hourly, midnight for daily)
5. YouTube and YouTube ReVanced share the same timer

## Time Windows

| Window | Reset Logic |
|--------|-------------|
| `/hour` | Resets every clock hour (2:00, 3:00, etc.) |
| `/6 hours` | Resets every 6 hours from midnight (0:00, 6:00, 12:00, 18:00) |
| `/day` | Resets at midnight |

## Supported Apps

| App | Package Name | Storage Key |
|-----|-------------|-------------|
| Instagram | `com.instagram.android` | `instagram` |
| YouTube | `com.google.android.youtube` | `youtube` |
| YouTube ReVanced | `app.revanced.android.youtube` | `youtube` (shared) |

## Key Files

- [AppTimeLimiter.kt](file:///C:/Users/Maaz/Projects/StayLocked/Reef/src/main/java/dev/pranav/reef/util/AppTimeLimiter.kt) — Singleton managing limits, usage tracking, and window resets
- [BlockerService.kt](file:///C:/Users/Maaz/Projects/StayLocked/Reef/src/main/java/dev/pranav/reef/accessibility/BlockerService.kt) — Accessibility service with app switch tracking + 30s poll
- [PowerXHomeScreen.kt](file:///C:/Users/Maaz/Projects/StayLocked/Reef/src/main/java/dev/pranav/reef/screens/PowerXHomeScreen.kt) — UI with FilterChip presets for each app
- [App.kt](file:///C:/Users/Maaz/Projects/StayLocked/Reef/src/main/java/dev/pranav/reef/App.kt) — Initialization

## Architecture

- **Storage:** SharedPreferences `"app_time_limits"` with keys like `instagram_enabled`, `instagram_limit_minutes`, `instagram_window`, `instagram_usage_ms`, `instagram_window_start`
- **Tracking:** `BlockerService` records which timed app is active and accumulates usage on app switches, screen off, and every 30s via a poll runnable
- **Window Reset:** Clock-aligned using `Calendar` — truncates to hour/6h/midnight boundaries
- **Lock-aware:** All UI controls disabled when `LockManager.isLocked()` is true

## Related Pages

- [[overview]]
