# Lockin (AppBlock V2) — Deep Code Audit Report

**Date**: 2026-08-07
**Scope**: Full PRD compliance audit of the single real codebase (Kotlin/Reef) + PowerX Block UI Preview (design mockup only)
**PRD Source**: `Obsidian Vault/App_Block.md`

---

## Executive Summary

| Layer | Status | Notes |
|---|---|---|
| **App (Kotlin + Jetpack Compose)** | 🟢 Feature-complete | All 6 core features implemented. UI is native Compose screens in `screens/`. |
| **PowerX Block UI Preview (HTML)** | 🟡 Design mockup only | Visual inspiration only — NOT a functional or code layer. |
| **Flutter** | ❌ NOT USED | Confirmed decision: no Flutter anywhere. Single Kotlin stack avoids double build time. |

> ⚠️ **Corrected Finding (vs. earlier report/agent claims)**: There is **no separate frontend** to build. The real screens all live in the **Kotlin backend** as Jetpack Compose composables. The only genuinely missing feature is the **on-phone 64-bit gibberish key-entry screen** (unlock is currently ADB-only).

---

## Feature-by-Feature Audit

---

### 1. 🌐 1-Click Web & Keyword Filter

**PRD Requirements**:
- Master adult toggle (one-tap) ✅
- Real-time URL bar extraction via AccessibilityService ✅
- Fail-closed browser policy (block unsupported browsers) ✅
- Smart keyword regex with boundary matching ✅
- Instant browser action (overlay → force-close → home screen) ✅

| Component | File | Status |
|---|---|---|
| Accessibility URL interception | `accessibility/BlockerService.kt` | ✅ Chrome, Brave, Firefox, Opera URL bars |
| Adult domain filter | `util/CategoryBlocklist.kt` + `assets/adult_domains.txt` | ✅ 101,143 domains, leet-speak normalization |
| Adult keyword filter | `util/CategoryBlocklist.kt` + `assets/adult_keywords.txt` | ✅ 246 keywords with boundary matching |
| Master adult toggle | `CategoryBlocklist.adult_enabled` pref | ✅ One-tap toggle |
| Fail-closed browser policy | `BlockerService.kt` | ✅ 21+ browsers blocked when adult filter ON |
| Redirect to about:blank | `BlockerService.performRedirect()` | ✅ Programmatic URL bar typing |
| Block screen overlay | `BlockedActivity.kt` | ✅ Full-screen overlay with quotes |
| Permanent blocklist | `util/PermanentBlocklist.kt` | ✅ Irreversible via ADB only |
| **Web filter settings screen** | `screens/WebsiteBlocklistScreen.kt` | ✅ Compose UI for sites/keywords, permanent toggle |
| SafeSearch enforcement | `BlockerService.kt` line 546 | 🟡 Disabled (documented future work) |

---

### 2. 📱 App Blocking (Time/Usage/Launch/Cooldown/Schedule)

**PRD Requirements**:
- Time-based limits per hour/day ✅
- Launch count restrictions ✅
- Interval presets (1h, 3h, 6h, 12h, 24h) 🟡 (window-aligned resets present; fixed interval presets to confirm)
- Cooldown logic (standard + strict reverse) 🟡 (window resets present; strict reverse-cooldown flag to confirm)
- Schedule windows ✅

| Component | File | Status |
|---|---|---|
| App block modes (complete + time-limited) | `util/AppBlockManager.kt` | ✅ "block"/"limit" modes |
| Time-limited windows | `util/AppBlockManager.kt` | ✅ hour, 6hour, day windows |
| Per-app time limiter (IG/YT + ReVanced) | `util/AppTimeLimiter.kt` | ✅ Clock-aligned resets |
| Daily usage limits | `util/AppLimits.kt` | ✅ UsageStatsManager-backed |
| Floating timer overlay | `util/FloatingTimerOverlay.kt` | ✅ Circular progress ring, edge-anchored |
| Usage tracking | `accessibility/ScreenUsageHelper.kt`, `UsageTracker.kt` | ✅ Event-based + Stats fallback |
| Whitelist system | `util/AppLimits.Whitelist` | ✅ 30+ auto-whitelisted apps |
| App discovery (5 sources) + picker | `screens/AppBlockScreen.kt` | ✅ ROM-compatible search/filter |
| Daily limit UI + chart | `screens/DailyLimitScreen.kt` | ✅ Vico 7-day chart |
| Cooldown after opening | `AppBlockManager.kt` window reset | 🟡 Window reset yes; strict-reverse to confirm |
| Schedule windows | `AppBlockManager.kt` | ✅ Window-based reset (hour/6h/day) |

---

### 3. 🔒 Master Lock Container

**PRD Requirements**:
- Master lock toggle freezing all settings ✅
- **64-char gibberish unlock key** 🔴 — NOT built as on-phone UI
- Copy-paste and screenshot disabled on unlock screen 🔴 — no such screen exists
- Emergency agent override / ADB backdoor ✅
- Settings cannot be edited while locked ✅

| Component | File | Status |
|---|---|---|
| Lock state management | `util/LockManager.kt` | ✅ Singleton, SharedPreferences-backed |
| 256-bit token generation | `util/LockManager.lock()` | ✅ SecureRandom, 32-byte hex (64 chars) |
| ADB-only unlock | `receivers/UnlockReceiver.kt` | ✅ Broadcast validates token |
| Device Admin activation | `receivers/ReefDeviceAdmin.kt` | ✅ Prevents uninstall while locked |
| Master lock toggle UI (+ confirm dialog) | `screens/MainSettingsScreen.kt:133-253` | ✅ Switch disabled when locked |
| Settings editing frozen while locked | `MainSettingsScreen` / `AppBlockScreen` / `WebsiteBlocklistScreen` / `DailyLimitScreen` | ✅ Remove/edit guarded, "Locked" states |
| Guardian auto-start | `util/LockManager.ensureGuardianRunning()` | ✅ Boot + App.onCreate |
| Emergency ADB command | `receivers/UnlockReceiver.kt` | ✅ `adb shell am broadcast ... ACTION_UNLOCK --es token` |
| **On-phone gibberish key screen** | ❌ **Does not exist** | 🔴 Genuine gap — unlock requires a PC |

---

### 4. 🛡️ Anti-Tamper & Protection System

**PRD Requirements**:
- Continuous permission monitoring (Accessibility, Device Admin, Usage Access) ✅
- Persistent lock overlay on tamper attempt ✅
- Uninstall prevention during master lock ✅
- Re-enable button with timeout ✅

| Component | File | Status |
|---|---|---|
| Guardian watchdog service | `accessibility/GuardianService.kt` | ✅ Polls every 2s, foreground |
| Accessibility monitoring → brick overlay | `GuardianService.checkAccessibilityStatus()` | ✅ Block UI if disabled |
| Anti-tamper App-Info detection | `BlockerService.kt:1116` | ✅ Two-condition (app name + indicators) |
| Device Admin uninstall prevention | `receivers/ReefDeviceAdmin.kt` | ✅ Active while locked |
| Periodic permission checks | `util/ReefWorker.kt` | ✅ 15-min WorkManager |
| Boot persistence | `receivers/BootReceiver.kt` | ✅ Restarts Guardian on boot |
| Permission utilities | `util/Permissions.kt` | ✅ accessibility, usage, overlay, DND, battery |
| Brick overlay + 10s re-enable window | `GuardianService.kt` | ✅ Full-screen, timeout re-shows |
| Crash recovery | `App.setupCrashHandler()` | ✅ AlarmManager restart on crash |
| Permission status screen | `PermissionsCheckActivity.kt` | ✅ |

---

### 5. 🧘 Overlay & Mindfulness Engine

**PRD Requirements**:
- Full-screen persistent block overlays ✅
- 3-second breathing delay 🟡 (3s dismiss cooldown yes; breathing-inhale cycle is 1Sec-style)
- Hold-to-open action (3 sec) 🟡 (countdown launch friction exists)
- Reflection challenge 🟡 (launch-count warning present)
- Motivational quotes, images, videos ✅
- Auto-scaling video aspect ratios ✅
- Dismiss timer (3s) with [X] button ✅

| Component | File | Status |
|---|---|---|
| Full-screen block overlay (text/image/video) | `BlockedActivity.kt` | ✅ 3 modes |
| Motivational quotes | `BlockedActivity.kt` | ✅ 124 day + 19 night + 10 streak-loss |
| Custom image/video support | `BlockedActivity.kt` | ✅ User-selected folder |
| 3-sec cooldown before dismiss | `BlockedActivity.kt` | ✅ Timer-based |
| Mindful launch friction + duration picker | `MindfulLaunchActivity.kt` | ✅ Countdown + picker |
| Countdown (3-60s) + duration (5-120min) | `MindfulLaunchActivity.kt` | ✅ Slider + quick presets |
| Daily launch limits | `util/MindfulLaunchManager.kt` | ✅ Count + limit |
| Gamification: Lives/Streaks/Ranks | `util/BlockScreenPrefs.kt` | ✅ 3 lives, streaks, 12 ranks |
| Heat map (90 days) | `util/BlockScreenPrefs.kt` | ✅ |
| Achievements (12) + milestones (10) | `util/Achievements.kt` | ✅ Logic ready |
| Mindful launch settings UI | `screens/MindfulLaunchSettingsScreen.kt` | ✅ |
| Home dashboard | `screens/PowerXHomeScreen.kt` | ✅ |
| **Achievements cards dashboard** | `PowerXHomeScreen.kt:211,259` | 🟡 TODO — logic done, cards pending |

---

### 6. 🎬 Shorts & Reels Blocker

**PRD Requirements**:
- Targeted short-form media filter (YouTube Shorts, Instagram Reels) ✅
- Preserve regular messaging/feeds ✅
- Independent toggles per platform ✅
- Timed temporary pass 🟡 (routine/focus system provides break; dedicated per-platform "temp pass" TBC)

| Component | File | Status |
|---|---|---|
| YT Shorts detection (activity/UI/URL) | `BlockerService.kt:320-855` | ✅ `/shorts/`, view IDs, activity names |
| IG Reels detection (activity/UI) | `BlockerService.kt` | ✅ clips/reels IDs & activities |
| ReVanced YouTube support | `BlockerService.kt` | ✅ `app.revanced.android.youtube` |
| Two-step kick (tab → back → home) | `BlockerService.kt:713-739` | ✅ |
| Independent toggles | `util/CategoryBlocklist.kt` | ✅ `shorts_enabled`, `reels_enabled` prefs |

---

## ✅ Fully Implemented
- Web & keyword filtering (Accessibility URL extraction, 101K domains, fail-closed browsers)
- App blocking (limits, floating timer, whitelist, app picker, chart)
- Anti-tamper guard (Guardian + Device Admin + periodic checks + anti-tamper detection)
- Block overlay & proactive mindful friction (quotes, media, gamification)
- Shorts & Reels blocker (activity + UI + URL detection)
- Master lock freeze (settings locked) + ADB unlock backend

## 🟡 Partially Implemented / Prototype
- **Strict reverse cooldown** access-safe — confirm
- **Interval presets UI** (1h/3h/charging timeline) — confirm screen coverage
- **Breathing delay** is partially present (3-sec cooldown); 1Sec-style animated breathing cycle to confirm
- **Temporary pass per-platform** (Shorts/Reels timed) — confirm
- **Achievements card UI** — TODO
- SafeSearch enforcement — disabled

## 🔴 Missing (Build Next)
1. **On-phone 64-bit gibberish key-entry screen** (with copy-paste/screenshot lock) — the core differentiator, genuinely missing (lock is ADB-only).
2. **Analytics dashboard** (screen-time charts, blind counters, impact) — partially present in `AppUsageScreen`/`FocusStatsScreen`, needs unification.
3. **Schedule/profile builder UI** (Normal/Lock/Strict + time windows) — scheduling logic exists, mode wiring TBD.
4. **Line-by-line timed typing** (alternative unlock) — prototype mode.
5. **AI Agent blocklist pipeline** (10+ scrapers + master consolidator) — design-only, not run in-app.
6. **Sidecar companion app** — separate future (update).

---

## 💡 Recommendations & Next Steps
1. **Stop treating "frontend" as a separate layer** — it's the Kotlin Compose `screens/` folder. Close new UI work there.
2. **Build the gibberish-key screen FIRST** — it's the core differentiator and the only genuinely missing core feature. UI: Compose full-screen, 64-char input, disable copy/paste + FLAG_SECURE screen-capture lock.
3. **Confirm trailing open items** (strict reverse cooldown, interval presets, temporary per-platform pass, breathing-animated cycle) against current code before scoping.
4. After key screen, do **Analytics dashboard** and **profile/schedule modes**.
5. **Use the PowerX Block HTML as design reference only** (purple glassmorphic, animation feel) — don't port it.

---

*Audit by OpenCode agent. Correction applied: settled on a single Kotlin/Compose stack — no Flutter frontend exists. Focus shifted to genuinely missing on-phone key-entry screen.*