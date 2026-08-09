# 🎯 Lockin — Feature Build Queue & Decisions

**Date created**: 2026-08-07
**Status**: Active build queue
**PRD source**: `Obsidian Vault/App_Block.md`

---

## 📋 Queued Features (In Order)

### 1. 🔑 Unlock System — Timer + Escalating Gibberish (REPLACES USB unlock)
- **Priority**: Highest
- **What**: Replace PC-USB (ADB/antigravity) unlock with two on-phone methods:
  - **Timer unlock**: pick a duration (e.g. 1 month) → no bypass until it elapses. Warning popup before locking.
  - **Escalating gibberish** (hard lock): type an exact random string (No copy-paste, FLAG_SECURE for screenshots).
- **Escalation logic (RESPECIFIED)**:
  - First unlock attempt → **64 characters**.
  - Each **wrong attempt** → size increases by **+10** (64 → 74 → 84 ...).
  - On a **later unlock event** (new lock session), start from the **last used size** (not the fixed 124), then continue **+10 per wrong**.
  - **Hard cap: 256 characters** (never above).
- **Remove**: USB debugging / antigravity ADB unlock entirely.
- **Status**: 🔴 Not built

---

### 2. 🎬 Shorts & Reels — Timed Pass (Presets)
- **Priority**: High
- **What**: When Shorts/Reels is blocked, offer preset pass options: **5, 10, 15, 20 min, 1 hour**. Auto re-locks when the timer expires.
- **No "Forever" option** — passes are always time-limited.
- **Hidden during Master Lock** — passes only show when lockdown is OFF. Strict mode = zero exceptions.
- **Status**: 🔴 Not built (targeted blocking already works)

---

### 3. 🏦 Credit System (Weekly Pool)
- **Priority**: High
- **Decision made**: ✅ **1 hour per week** (refill every week). No "Forever" option.
- **Reset rule**: At the next weekly cycle, the pool resets to 1 hour — **no carryover** of unused credits.
- **How it works**:
  - User gets a **1-hour credit budget per week**.
  - One shared pool at the top — can be used on any app, all at once or a little at a time.
  - Pass presets: **5, 10, 15, 20 min, 1 hour** — each spends the corresponding credit amount.
  - When a block triggers (e.g. hit the "5 min per hour" limit), spend credit to extend.
  - Extension presets: **1 min, 5 min, 1 hour**.
  - **Session rule**: Credit may be spent only **once per block window (that 1-hour session)**.
  - A **new session = the 1-hour block reset** (auto-refreshes the 5-min limit). It does **NOT** require reopening the app.
  - Once the period's credit is empty → wait until the next period for it to refill.
- **Why**: Acts as a "escape hatch" but forces the user to budget it — matches the intentional-friction goal. Hard to bypass since it's tied to period, not app restarts.

---

### 4. 🔄 Reverse Cooldown (gradual refill, no reset to zero)
- **Priority**: High
- **What**: After a per-app limit is hit (e.g. 5 min/hour), the allowance refills **gradually over time** — NOT a snap back to full.
- **Refill gate**: Refill only starts after being away **2–3 min** (kills the 1-second close/reopen exploit).
- **Visual**: Small cooldown gauge on block screen, e.g. "Timer cooldown: 40% (2 min earned). Stay away to refill."
- **Terminology rule**: Use **"timer cooldown"** wording — NO battery/charging metaphors.
- **Not doing**: "Streak Bonus" idea (reintroduces gamification — conflicts with Step 5).
- **Status**: 🔴 Not built (refines the old "strict reverse cooldown" open item)

---

### 5. ⚡ Quick Block Session
- **Priority**: Medium/High
- **What**: One-tap "block these apps right now for X minutes" from the home screen.
  - Pick presets (e.g. 30 min) + pick what to block → blocks instantly → auto-expires.
- **What's missing today (why it's not on this home screen already)**:
  - No fire-and-forget quick button on home — blocking today requires going into deep screens (Routines/Routine create flow).
  - Full/clean markup: need a fast modal on home: time preset + app picker + instant-enable.
  - Note: quick options already exist in `MindfulLaunchActivity` + routine presets, but not wired as an instant "block now" session.
- **Status**: 🔴 Not fully built (core blocking exists, but not as a dedicated quick-session flow)

---

### 6. 🗃️ Archive Gamification (Lives/Streaks/Ranks/Achievements/Heat map)
- **Priority**: Medium
- **What**: Remove/archive the gamification layer from the app (not force on new users).
- **Approach**: Archive (don't delete) so the copy + logic is retrievable for future. Code is scattered across `BlockScreenPrefs.kt`, `Achievements.kt`, `PowerXHomeScreen.kt` (streak/level/milestone cards), `BlockedActivity.kt` (lives).
- **Status**: 🔴

---

### 7. 🛡️ Watchdog — Smarter Permission Monitoring (poll → event-driven)
- **Priority**: Medium
- **Current**: `GuardianService.kt` polls accessibility status every 2s (Handler.postDelayed, 2000ms).
- **Goal**: Reduce polling; prefer listening for change events + slow fallback poll (30-60s) instead of 2s.
- **Status**: 🔴 Design confirmed ✓ (See [[watchdog-response]] for Hybrid Event-Driven Spec)

---

### 8. 🌐 Browser Registration / Fail-Closed (per user's decision)
- **Priority**: Medium
- **Confirmed approach**: Register a browser package from within the app.
  - If we can **read** its URL bar → filter it.
  - If we **can't read** it → treat as unknown → **block entirely** (fail-closed).
- **Current**: 4 supported browsers (Chrome/Brave/Firefox/Opera) + 21+ explicitly blacklisted. Need a "register new browser" UI that adds a package and auto-classifies read/block.
- **Status**: 🔴 Adding registration UI

---

## 📌 Open Items / Unconfirmed (to verify before building)
- Interval presets UI (1h/3h/6h/12h/24h) — confirm screen coverage.
- 1Sec-style animated breathing cycle (vs. fixed 3-sec cooldown).
- Temporary pass / break system for other blocks, not just Shorts/Reels.
- Analytics dashboard — unify `AppUsageScreen` / `FocusStatsScreen` into one.
- SafeSearch enforcement — currently disabled (`BlockerService.kt:546`).

---

## ✅ Already Implemented (for reference)
- Web & keyword filter (Accessibility URL extraction, 101K+ domains, fail-closed browsers)
- App blocking (limits, floating timer, whitelist, app picker, chart)
- Master lock freeze + ADB unlock backend
- Anti-tamper guard (Guardian + Device Admin + periodic checks)
- Block overlay & mindful friction (quotes, media, gamification)
- Shorts & Reels **targeted blocking** (activity + UI + URL detection) — ONLY the 5-min pass is missing

---

## 🎨 Stack & Design notes
- **No Flutter** — single Kotlin + Jetpack Compose stack (avoids double build time).
- UI screens live in `screens/` (Compose). Backend = native Kotlin.
- Use `PowerX_Block_UI_Preview` (HTML) **only as visual reference** (purple/violet glassmorphic feel) — don't port it.

---

*Log maintained in the Obsidian Vault. Update statuses as tasks are completed.*