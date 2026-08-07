---
type: prd
status: active
summary: Master Product Requirement Document for Lockin (Flutter UI + Native Kotlin Hybrid)
last_updated: 2026-08-07
---

# 🚀 Lockin — Master Product Specification

> **Overview**: An anti-bypass, highly customizable Android productivity app designed to combat phone addiction, block adult content, and eliminate social media media distractions (YouTube Shorts / Instagram Reels) using un-bypassable lock rules, AI-driven blocklists, and friction-based mindfulness overlays.

---

## 🎯 1. Executive Vision & Architecture

* **Primary Goal**: Combat phone & adult content addiction through un-bypassable locking rules, intentional friction, and smart filtering.
* **Key Differentiator**: Solves the core flaw of existing market apps (like StayFocusd) by eliminating bypass loopholes (preventing permission disabling, force stops, or settings tampering) while offering deep customization.
* **Tech Architecture**: Hybrid Flutter (Frontend / UI, 120fps) + Native Kotlin (Backend Services, Accessibility Service, Device Admin API).
* **Reference Codebase**: Native Kotlin reference files located at `C:\Users\Maaz\Projects\StayLocked\Reef\` (`AppBlockManager.kt`, `BlockerService.kt`, `AppBlockScreen.kt`).

---

## 🔒 2. Anti-Bypass & Anti-Tamper Security

### 🛡️ Anti-Tamper Guard
* **Continuous Permission Monitoring**: Continuously monitors Accessibility, Device Admin, and Usage Access permissions.
* **Persistent Lock Overlay**: If a user attempts to disable permissions or tamper with settings, an immediate full-screen overlay blocks phone usage except for a single **"Fix Settings"** button directing back to settings.
* **Uninstallation Lock**: Prevents uninstalling the app while an active master lock container session is running.

### 🔑 Advanced Unlock Mechanisms
* **64-Bit Gibberish Password**:
  * Requires manually typing an exact, case-sensitive 64-character random string.
  * **Copy-paste and screenshot capture are strictly disabled** on this screen.
* **Line-by-Line Timed Typing (Alternative / Prototype Mode)**:
  * Displays text line-by-line with timed delays (e.g., Line 1 visible for 10–30 seconds before switching to Line 2).
  * Prevents memorizing or taking single screenshots of the full unlock key.
* **Emergency Agent Override**: Option for emergency unlock via AI agent / admin backdoor if configured.

---

## 🌐 3. Smart Web & Keyword Blocking (AI Pipeline)

### 🚫 Website & Adult Content Filter
* **Master Adult Toggle**: One-tap filter blocking adult websites, domain extensions, and related search terms.
* **Supported Browser Inspection**: Real-time URL bar extraction via Kotlin `AccessibilityService` (Chrome, Edge, Brave, Firefox, etc.).
* **Fail-Closed Browser Policy**: Any browser where the URL bar cannot be inspected (or secret browsers like Tor trying to bypass filters) is **instantly blocked from opening**.
* **Smart Keyword Regex**: Boundary matching (`\bkeyword\b`) to eliminate false flags on safe words (e.g., *"Sussex"*, *"Human Biology"*).
* **Instant Browser Action**: When a restricted URL or keyword is detected:
  1. Full-screen overlay blocks view immediately.
  2. The browser tab is force-closed.
  3. Sends user to Home Screen and prunes browser from recent tasks to prevent 3-button navigation loops.

### 🤖 10+ AI Agent Multi-Agent Pipeline
1. **Data Harvesting (Agents 1–10)**: 10 parallel AI agents scrape open-source repos, DNS blocklists (AdGuard, StevenBlack), and GitHub lists into separate files.
2. **Master Consolidation (Agent 11)**: Deduplicates, cleans, and formats all gathered chunks into a unified master blocklist.
3. **Architect Review Agent**: Audits the list for false positives and flags questionable entries.
4. **Manual Additions**: User can manually append custom URLs and keywords.

---

## 📱 4. Granular App Blocking & Flexible Rules

### ⚙️ Per-App Custom Rules
* **Time-Based Limits**: Set usage limits per hour/day (e.g., max 5 mins usage every 1 hour).
* **Launch Count Restrictions**: Limit total app launches per day.
* **Interval Presets**: Lock durations (1h, 3h, 6h, 12h, 24h).
* **Cooldown Logic**:
  * *Standard Cooldown*: Timer runs continuously.
  * *Strict Reverse Cooldown*: Opening the app during cooldown resets the timer back to zero.

### 🔒 Master Container Lock
* Activating the **Master Container Lock** freezes all individual app/site settings. Rules cannot be edited or turned off until unlocked via the gibberish key.

---

## 🎬 5. Shorts & Reels Friction & Selective Blocking

* **Targeted Media Filter**: Specifically blocks short-form media feeds (YouTube Shorts, Instagram Reels) while preserving regular messaging and feeds (e.g., watching main YouTube videos, Instagram DMs).
* **Timed Temporary Pass**: Option to temporarily unblock Reels/Shorts for a set time (e.g., 5 minutes), after which auto-lock re-engages.
* **Independent Toggles**: Separate controls for YouTube Shorts vs. Instagram Reels.

---

## 🧘 6. Mindfulness Delay & Intentionality Overlays

* **Breathing Delay (Inspired by 1Sec)**: Opening a blocked app triggers a 3-second breathing overlay (*"Take a deep breath"*).
* **Hold-to-Open Action**: Requires holding down an opening button for 3 seconds to confirm intent.
* **Reflection Challenge**: Prompts a short quote or reflection code to type before entry, breaking automatic muscle-memory habits.

---

## 🎥 7. Motivational Media Engine on Block

* **Dynamic Block Overlay**: Displays motivational content instead of standard error messages:
  * Inspiring quotes & affirmations.
  * Short motivational video clips / reels.
* **Media Sources**: Loaded from local app directory (`/Engine/Media`), custom user folders, or cloud storage.
* **Smart UI Fit**: Auto-scales video aspect ratios to fit any mobile screen dynamically without stretching.
* **Dismiss Timer**: Displays a discreet `[X]` close button after 3 seconds to return to the Home Screen.

---

## 📅 8. Scheduling & Profile Modes

* **Time Schedules**: Define active blocking windows (e.g., Locked 12:00 PM – 6:00 PM, Unlocked 6:00 PM – 12:00 AM).
* **Profile Modes**:
  * **Normal Mode**: Basic reminders and soft limits.
  * **Lock Mode**: Enforces active time schedules.
  * **Strict Mode**: Full anti-bypass active; zero modifications permitted.

---

## 📊 9. Analytics & Impact Dashboard

* **Visual Metrics**: Charts showing daily, weekly, and monthly screen time breakdowns.
* **Block Counters**: Track total blocked attempts for websites, apps, and Shorts/Reels.
* **Impact Tracking**: Compares average screen time before app installation vs. current usage to track progress.

---

## 🛡️ 10. Anti-Tamper Sidecar Companion App
* Standalone lightweight background service app that enforces permissions and anti-tamper overlays for both **Lockin** and third-party apps like StayFocusd.

---

## 🎨 11. UI/UX Design System
* **Color Palette**: Sleek dark mode with **purple / violet accent theme** (modern alternative to basic blue designs).
* **Interface Style**: Dynamic glassmorphic cards, Google Fonts typography (Inter / Outfit), fluid micro-animations.

---

## ❓ 12. StayFocusd Feature Clarification: "Take a Break"
* **Function**: StayFocusd's "Take a Break" feature acts as a quick pause for active blocks (30 mins, 1 hour, 4 hours, 1 day). When time expires, locks re-engage automatically.
* **Lockin Implementation**: Integrates into the *Temporary Pass* system, enforcing mindfulness delays before granting a break.
