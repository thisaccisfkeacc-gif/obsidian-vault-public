# 🎯 Lockin — Feature Build Queue & Decisions

**Date created**: 2026-08-07
**Status**: Active build queue
**PRD source**: `Obsidian Vault/App_Block.md`

---

## 📋 Queued Features (In Order)

### 1. 🔑 64-Bit Gibberish Key Screen
- **Priority**: Highest
- **What**: On-phone full-screen unlock screen. User must type an exact 64-character random string.
- **Extra**: Disable copy-paste + block screenshots (FLAG_SECURE).
- **Why now**: Current unlock requires a PC (ADB-only). This removes that dependency — the core differentiator.
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

## 📌 Open Items / Unconfirmed (to verify before building)
- Strict reverse-cooldown flag (opens resets timer) — confirm present in code.
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