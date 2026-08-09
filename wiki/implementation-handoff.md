# 🛠️ Lockin — Implementation Handoff Report (for the coding agent)

> **Role**: You are the implementing agent. This document is your spec. Read it fully, then make ALL the code changes described below, in order. Verify your work builds before finishing (or tell the caller what to check).

## Project Location & Stack
- **App**: Kotlin + Jetpack Compose. UI lives in `screens/`. Backend = native Kotlin.
- **App root**: `C:\Users\Maaz\Projects\StayLocked\Reef\`
  - Main package: `src/main/java/dev/pranav/reef/`
  - Key services: `accessibility/BlockerService.kt`, `accessibility/GuardianService.kt`, `receiver/UnlockReceiver.kt`, `receiver/ReefDeviceAdmin.kt`
  - Key managers: `util/LockManager.kt`, `util/AppBlockManager.kt`, `util/AppTimeLimiter.kt`, `util/CategoryBlocklist.kt`, `util/BlockScreenPrefs.kt`, `util/Achievements.kt`
  - UI: `screens/MainSettingsScreen.kt`, `screens/AppBlockScreen.kt`, `screens/PowerXHomeScreen.kt`, `screens/WebsiteBlocklistScreen.kt`
- **No Flutter anywhere.** Do NOT scaffold a Flutter app. Extend the existing Kotlin app.

---
**IMPORTANT RULES**
- Do not edit `.ahk` scripts (not applicable here).
- Follow Step order. Do not skip.
- After each change, confirm it builds before the next where possible.
- Keep files stylistically consistent with existing code (Services + Singleton managers + Compose screens).
- Settings prefs are stored in a **device-protected** context (`createDeviceProtectedStorageContext()`); reuse that pattern.

---

## STEP 1 — Unlock System (REPLACES USB/ADB unlock) | Priority: Highest

**Goal**: Remove USB/ADB unlock. Add on-phone unlock with spentum PLUS escalating gibberish.

### 1A. Timer-based unlock
- When locking, allow choosing a **duration** (hours / days / months). During that window, unlock is NOT allowed (no bypass).
- Show a **warning popup** before locking confirming the duration and "no unlock before it expires."

### 1B. Escalating gibberish key (replaces ADB token unlock)
- On-phone full-screen unlock screen. User must type an exact random string.
- No copy-paste + `FLAG_SECURE` (block screenshots) on the unlock screen.
- **Escalation rules (exact):**
  - 1st unlock attempt → **64** characters.
  - Each **wrong attempt** → size increases by **+10** (64 → 74 → 84 ...).
  - On a **later unlock event** (a NEW lock session), start from the **last used size** (store it; e.g. if last was 64, next session starts at 64, then +10 per wrong). It does NOT restart at 64 each session.
  - **Hard cap: 256** characters (never above).
- Keep storing the current valid key in device-protected prefs (`LockManager`).

### 1c. Remove USB/ADB unlock
- Remove / stop exposing the ADB `UnlockReceiver` broadcast unlock path as the primary mechanism.
- Decide: either remove it entirely, or keep it strictly as a debugging fallback gated behind `BuildConfig.DEBUG`. Recommend keeping a debug-only fallback; remove from production path.
- Update `LockManager.unlock()` to the new tamper methods + remove calls that referenced the broadcast token unlock from user flows.

### Files to change
- `util/LockManager.kt` — lock/unlock logic, duration, escalation state, last-used size.
- `UnlockReceiver.kt` — remove / debug-gate.
- NEW or updated: an unlock Compose screen (`screens/`) that implements the escalating-gibberish screen + timer display.

---

## STEP 2 — Shorts & Reels Timed Pass

**Goal:** When Shorts/Reels is blocked, let the user take a timed pass.

- **Preset options** when blocked: **5, 10, 15, 20 min, 1 hour**.
- Auto re-lock when the timer ends.
- **No "Forever" option.**
- **Hidden during Master Lock** — when the master lock is ON, do NOT show the pass screen (zero exceptions).
- **UI/UX rule (applies to ALL block screens):** The dismiss / pass button on the block screen must be **a compact ICON button** (icon only, no text label) tucked in a corner — deliberately NOT attention-seeking. No big glowing text buttons that pull the eye toward "skip." Keep it small, subtle, and out of the way.

**Where:** Shorts/Reels detection & kick already lives in `BlockerService.kt` (`kickFromShortsReels`, `isShortsBlocked`, `isReelsBlocked`). Add a pass timer store + UI to grant.

---

## STEP 3 — Credit System

**Goal:** A weekly pool that price sits on top of the pass feature.

- **Budget: 1 hour per week.**
- **Reset rule (exact):** At the start of the next weekly cycle, the pool **resets to 1 hour and does NOT carry over** any remaining credits. Unused credits are lost.
- One shared pool (can be used on any app, all-at-once or bit-by-bit).
- Preset spending caps (minor covered in Step 2): the pool value is 60 min/week total across sessions.
- **Session rule (exact):**
  - Credit may be spent only **once per block window (that 1-hour session)**.
  - A **new session = the 1-hour block reset** — the block auto-resets the app's own 5-min-per-hour limit. It does NOT require reopening the app / a viewer.
  - So: prevent spending credit twice within the same ongoing block window even if the user relaunches; the reset of the app's block timer (the 1-hour session) is what unlocks the next spend.
- When the weekly credit is empty → locked until next week.
- Tie into Step 2 pass presets so each pass spends from the pool.

---

## STEP 3.5 — Reverse Cooldown (gradual refill, no reset to zero)

**Goal:** Fix how the per-app allowance recharges after the limit is hit.

**Current behavior (wrong):**
- App has a limit (e.g. 5 min per hour). Once it's hit, the user is on the block screen and must wait a full hour; closing/reopening just restarts the same wait. There's no smart recharging.

**New behavior (implement this):**
- When the user **closes the app**, the allowance begins refilling **gradually over time** — it does NOT snap back to a fresh full amount.
- If they reopen shortly after, they get back **only the time that has been refilled so far** (a small amount), not a fresh full allowance.
- The refill is gradual, so opening/closing the app repeatedly does **not** exploit a free full reset. The only path to a full allowance is real physical time passing.

**Refill gate (anti-exploit):** Refill does NOT start until the user has stayed away from the app for a minimum window (e.g. **2–3 minutes**). This prevents 1-second close/reopen cycles from farming tiny crumbs.

**Visual on block screen:** Show a small live indicator on the block screen, e.g. a **cooldown gauge** — *"Timer cooldown: 40% (2 min earned). Stay away to recharge fully."*
- ⚠️ **Terminology rule:** Use **"timer cooldown"** wording, NOT battery/charging metaphors. Users should read it as a cool-down timer, not a phone battery.

**What we explicitly chose NOT to do:** The other agent's suggestion of a **"Streak Bonus"** (stay away 3h → refill 2x faster) is **skipped** — it reintroduces gamification, which we are archiving in STEP 5. Can revisit later if un-archived.

- Replaces the earlier "strict reverse cooldown" open item — this is the refined, user-friendly version.

---

## STEP 4 — Quick Block Session

**Goal:** One-tap "block now for X minutes" from home.

- Home-screen button → fast modal: pick a **time preset** + pick **what to block** (apps and/or website groupings / Shorts & Reels ).
- Blocks instantly, auto-expires.
- Requirements: must not depend on deep navigation; must set a quick temp-block that respects existing managers.

---

## STEP 5 — Archive Gamification (Likes/Streaks/Ranks/Achievements/Heat map)

**Goal:** Remove from the active UI, keep for future.

- Approach: **Archive, do NOT delete.** Make the gamification layer optional/disabled, or relocate it behind a flag so it's not shown to new users; keep the source retrievable.
- Affected code:
  - `util/BlockScreenPrefs.kt` (lives/streaks/heat map)
  - `util/Achievements.kt`
  - `BlockedActivity.kt` (lives)
  - `PowerXHomeScreen.kt` (streak / level / milestone cards)
- Recommendation: keep files, gate off their on-screen use (e.g. a `ENABLE_GAMIFICATION` flag defaulting to OFF), and remove their entry points from home.

---

## STEP 6 — Smarter Watchdog (event-driven instead of 2-sec poll)

**Approved hybrid design (do this):**
- **Primary:** use a real listener:
  - `AccessibilityManager.addAccessibilityStateChangeListener { … }` to react instantly when the accessibility enabled state changes, AND/OR a `settings.Secure` `ContentObserver` on `ENABLED_ACCESSIBILITY_SERVICES`.
  - Show brick overlay immediately (< ~100ms) when accessibility is toggled off while locked.
- **Adaptive fallback polling:**
  - **Screen ON & user in Settings:** poll aggressively (~500ms).
  - **Screen ON & normal use:** slow heartbeat (~30–60s).
  - **Screen OFF:** pause polling (0 battery).
- Keep prevent** uninstall via Device Admin (`ReefDeviceAdmin.kt`), and consider `dpm.setUninstallBlocked()` if an admin is active.
- **Re Screen** - the user emphasized: do NOT block the whole Settings app. Only intercept when the user tries to disable OU accessibility service / modify the target block — normal settings use stays open. Implementation in `BlockerService.kt` already hints at detecting our App Info page; keep it targeted.
- Update `GuardianService.kt` `CHECK_INTERVAL_MS` / state machine accordingly.

---

## STEP 7 — Browser Registration / Fail-Closed (GUI)

**Approved approach:** Register a browser package from within the app.
- If we can **read** its URL bar → treat as readable → filter blocklists normally.
- If we **can't read** it → treat as unknown → **block entirely** (fail-closed).
- Currently: 4 supported readable browsers + 21+ in a hard block list in `BlockerService.kt`. Add a **"add browser" UI** that registers a package and auto-classifies read vs blocked. Store the registration (device-protected prefs).

---

## After Implementation — Verification Checklist
1. Build: `dotnet `<not applicable> — use `cd <app>` then `./gradlew compileDebugKotlin` (or the project's normal build) to compile Kotlin.
2. Run a `MainActivity` / service registration sanity (the app must start without crashes).
3. Run smoke: every Compose screen referenced still compiles.
4. Report per Step: Done / Wait / error the caller needs to fix.

---

## Report Format You Must Return (at the very end)
Return **Findings → Recommendation → Done**, per step:
- For each of Steps 1-7: ✅ DONE / 🟡 PARTIAL / 🔴 BLOCKED + one line what was changed + one line what's left.
- Any build errors you could not fix.
- Files changed list.

---

### Final Notes
- This is a single Kotlin app (no Flutter). **Do not** add Flutter.
- Save ideas/log later; focus only on what's here.
- If a requirement conflicts with existing intended design, flag it in your report and stay faithful to this doc.

---
*Handoff spec written 2026-08-07. Follow order 1→7.*