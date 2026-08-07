---
type: prd
status: active
summary: Master Product Requirement Document for Lockin (Pure Native Kotlin Android)
last_updated: 2026-08-07
---

# 🚀 Lockin — Master Product Specification

> **Overview**: An anti-bypass, highly customizable Android productivity app designed to combat phone addiction, block adult content, and eliminate social media media distractions (YouTube Shorts / Instagram Reels) using un-bypassable lock rules, AI-driven blocklists, and friction-based mindfulness overlays.

---

## 🎯 1. Executive Vision & Architecture

* **Primary Goal**: Combat phone & adult content addiction through un-bypassable locking rules, intentional friction, and smart filtering.
* **Key Differentiator**: Solves the core flaw of existing market apps (like StayFocusd) by eliminating bypass loopholes (preventing permission disabling, force stops, or settings tampering) while offering deep customization.
* **Tech Architecture**: Pure Native Kotlin (Jetpack Compose UI + Backend Services, Accessibility Service, Device Admin API).
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




NEW IDES

# App Concept: Advanced Blocking & AI-Controlled Lockdown System

The app should let users block distracting content with a single tap. This includes adult websites, adult-related keywords, and any manually added websites or keywords.

For manually added websites, the blocking should be configurable on a per-site basis. Not every manually added website is necessarily inappropriate—for example, users may want to limit access to movie or anime websites instead of blocking them completely. Each website should therefore have its own rules, such as complete blocking, time limits, schedules, or other restrictions. Manual keywords should also be supported with similar flexibility.

In the future, there may be situations where a legitimate search is accidentally blocked because it resembles a restricted keyword. To solve these edge cases, the app can eventually include a built-in AI agent. This agent will not simply approve every request—it should act like a careful investigator. It should ask for the full context, request explanations, ask follow-up questions, and even accept text or images as evidence before making a decision. It should be difficult to deceive, and it should reject requests whenever the explanation is inconsistent or unconvincing.

The AI agent should also be able to access the internet when necessary to verify information. However, this feature is planned for the future. The priority is building a stable and reliable blocking system before introducing advanced AI capabilities.

## Lockdown Mode

Once Lockdown Mode is enabled, users should not be able to remove blocked websites, keywords, or restrictions through normal app controls.

Initially, the only ways to unlock the app should be:

- **Developer emergency unlock** (via a trusted recovery method such as USB debugging or another secure developer-controlled mechanism).
    
- **Future AI agent unlock**, where the AI evaluates whether the unlock request is genuinely justified.
    

Eventually, if the AI agent becomes reliable enough, the developer recovery method could be removed entirely.

The AI should treat every unlock request as a last resort. It should require a complete explanation, ask follow-up questions, request supporting evidence when needed, and refuse requests that appear dishonest or inconsistent. Unlocking should only happen when there is a genuinely valid reason—not simply because the user wants to bypass their own restrictions.

This is especially important because users usually enable Lockdown Mode to protect themselves from addiction. At that point, their future self should not be able to easily override the decision made by their past self.

## Long-Term AI Vision

Over time, the AI should evolve beyond being just an unlock assistant. It should become the central controller of the app.

Eventually, it should be able to:

- Understand user analytics and behavior.
    
- Monitor long-term usage patterns.
    
- Make informed decisions based on context and history.
    
- Manage restrictions intelligently, even when the Master Lock is disabled.
    
- Effectively act as the brain of the application.
    

## Alternative Unlock Method

A second unlock method could be a **64-character gibberish challenge** (or another extremely difficult recovery challenge).

This creates two independent recovery paths:

1. **AI Unlock:** The user explains the situation to the AI, which evaluates the request and unlocks the app only if it is genuinely justified.
    
2. **Challenge Unlock:** The user completes the difficult recovery challenge instead of speaking to the AI.
    

An interesting idea is to combine these two systems. If the user chooses the AI route, the AI has access to the full conversation and context. If the user chooses the challenge route instead, the AI has no background information, so after the challenge is completed it could still ask questions or perform additional verification before allowing the unlock.

This layered approach would make bypassing Lockdown Mode significantly more difficult while still providing legitimate recovery options for genuine emergencies.



IDEA 2

### AI-Controlled Lockdown System (Idea Summary)

The long-term vision is to make the AI agent the brain of the app. Instead of being just a chatbot, it will have permission to control app settings, modify limits, and approve or reject unlock requests.

The AI will **not** be part of the first release. First, the app should become completely stable, with every core feature implemented and thoroughly tested. Only after that will the AI system be developed and integrated.

### Unlock Logic

There are two levels of unlocking:

- **Per-app unlocks** (increase limits, temporary access, etc.) — easier and less strict.
    
- **Master Lock unlocks** (disable Lockdown Mode) — much more difficult.
    

For Master Lock, the AI should require a complete explanation, ask multiple follow-up questions, and only proceed if it is fully convinced that the reason is genuine.

Even after approval, the user must complete an additional challenge, such as typing a 64-character random string. The order can be either:

1. Explain the situation → AI approves → Complete the challenge.
    
2. Complete the challenge → Explain the situation → AI gives final approval.
    

Both approaches are possible and can be evaluated later.

### Progressive Strictness

The system should become stricter over time.

- First unlock request → relatively easy.
    
- Second unlock request → stricter, with more questions.
    
- Third and later requests → significantly harder, with clear warnings before proceeding.
    

The AI should remember previous unlocks, including why they happened. If the user repeatedly unlocks without following through on the stated reason, future requests become increasingly difficult.

The recovery challenge can also become harder over time:

- 64 characters
    
- 128 characters
    
- 256 characters (maximum)
    

### Time-Based Strictness

The AI's strictness should depend on how much access the user requests.

For example:

- **5-minute extension** → minimal questioning.
    
- **30–60 minutes** → moderate questioning.
    
- **12–24 hours** → much stricter verification.
    

Longer unlock durations require stronger justification.

### Different Rules for Different Actions

Increasing the limit for a single app should be easier than disabling the entire Master Lock.

If the user only needs temporary access to one app for work or another valid reason, the AI should allow it with lighter verification.

Disabling Lockdown Mode entirely should always require the highest level of scrutiny.

### Development Plan

The AI should only be built after the app is feature-complete and stable.

Before integrating it into Android, the AI logic should be tested inside a custom HTML simulation where different scenarios can be tested quickly without rebuilding the mobile app every time. Once the AI behaves correctly, it can then be integrated into the Android application.

### Research Before Implementation

Before implementing this system, multiple AI agents should independently evaluate the concept.

Each agent should provide:

- Feasibility analysis.
    
- Security concerns.
    
- Weaknesses.
    
- Improvements.
    
- Better logic.
    
- Alternative approaches.
    

All responses should be collected into one document, compared, and the best ideas should be incorporated into the final design.

### Possible Future Unlock Methods

These are only ideas and are **not finalized**:

- Email OTP verification.
    
- Phone OTP verification (if feasible).
    
- Pay-to-Unlock (user pays a fixed amount through QR payment to discourage impulsive unlocking).
    
- Other secure recovery methods if they provide genuine value.
    

These ideas may or may not be implemented, but they should remain on the brainstorming list for future evaluation.

Overall, the goal is to create an AI that behaves like a strict accountability partner rather than a simple assistant. It should make impulsive unlocking extremely difficult while still allowing genuine emergencies to be handled fairly.