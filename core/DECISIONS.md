---
purpose: Agent Decision Records – Architectural Audit Log
project: PowerX Keys
version: 1.1
date: 2026-07-09
instructions: >
  READ THIS before making any major architectural or design decision.
  APPEND a new AgDR entry when you make a significant choice.
  Never delete entries – only mark as superseded.
  "Significant" = a choice that future agents might accidentally reverse.
---

# 📋 DECISIONS.md – Agent Decision Records (AgDR)

> **Why this exists:** Agents are stateless. Without this file, every new agent rediscovers the same architectural trade-offs and sometimes makes conflicting choices. This file is the permanent, auditable record of *why* things are the way they are.

## AgDR Template

```markdown
## AgDR-NNN – [Short Decision Title]
- **Date:** YYYY-MM-DD
- **Context:** What problem triggered this decision?
- **Decision:** What was chosen?
- **Rationale:** Why?
- **Consequences:** What does this lock in?
- **Status:** active | superseded-by-AgDR-NNN | deprecated
```

---

## Decision Log

## ADR-0001 – White-Labeled AutoHotkey v2 Execution Engine
- **Date:** 2026-07-28
- **Context:** Low-level Windows macro execution requires sub-millisecond input hook and pixel search performance.
- **Decision:** C# JSON config is compiled by `ScriptCompilerService.cs` into AHK v2 scripts executed via `PowerX_Engine.exe`.
- **Rationale:** Native AHK v2 handles GDI pixel search and Windows low-level input hooks with maximum speed. Never edit `.ahk` files directly.
- **Status:** active

## ADR-0002 – SQLite Local Database Storage
- **Date:** 2026-07-28
- **Context:** Need zero-latency, offline-first macro and settings persistence.
- **Decision:** Macros and triggers stored in local SQLite (`%LOCALAPPDATA%/PowerXKeys/macros.db`). App config in `AppConfig.json`.
- **Rationale:** Ensures zero latency on global hotkey trigger without network dependency.
- **Status:** active

## AgDR-001 – URL-Only Validation for File Launcher Gear Menu
- **Date:** 2026-07-15
- **Context:** The gear menu for File Launcher blocks had a combined "Path or URL" input that accepted both file paths and URLs.
- **Decision:** Restricted the input to URLs only (http/https/ftp). File paths are handled via the Browse button only.
- **Rationale:** Prevents invalid AHK scripts from being generated when a non-URL string is saved as a launcher path.
- **Consequences:** The gear textbox will reject file paths with a warning. File paths must be set via Browse.
- **Status:** active

## AgDR-002 – URL Settings Section Hidden by Default
- **Date:** 2026-07-15
- **Context:** The URL Settings section in the File Launcher gear menu was causing confusion since the block now also supports file paths via Browse.
- **Decision:** Wrapped the URL Settings section in a `StackPanel` with `Visibility="Collapsed"`. A dev flips it to `Visible` to show it.
- **Rationale:** Keeps the gear menu clean. URL input is a dev-controlled feature, not a user-facing one by default.
- **Consequences:** Changing visibility requires editing the XAML directly.
- **Status:** active

## AgDR-003 – MinimizeIfActive Added to App Switcher
- **Date:** 2026-07-15
- **Context:** App Switcher had no way to minimize a window that was already active.
- **Decision:** Added `MinimizeIfActive` bool property to `ActionItem`. When true, AHK checks `WinActive()` first and calls `WinMinimize()` instead of activating.
- **Rationale:** Common "toggle window" UX pattern — press once to focus, press again to minimize.
- **Consequences:** Works in both standard and `CycleThroughWindows` paths.
- **Status:** active

## AgDR-004 – Toast Notification Strips ahk_exe and .exe
- **Date:** 2026-07-15
- **Context:** Capture notifications showed raw identifiers like `Bound to Ready To Help ahk_exe Antigravity.exe!` which looked messy.
- **Decision:** Strip `ahk_exe` and `.exe` from the identifier string before displaying in `ToastMessage`.
- **Rationale:** Cleaner, more user-friendly notification text.
- **Status:** active

## AgDR-005 – Block Card Display Strips URL Prefixes and File Path Prefix
- **Date:** 2026-07-15
- **Context:** Block cards showed full URLs and file paths with `...\` prefix, making them hard to read.
- **Decision:** URLs strip `http(s)://` and `www.` then middle-truncate. File paths show filename only (no `...\` prefix), also middle-truncated.
- **Rationale:** Cards have limited space. Showing the meaningful part only is cleaner.
- **Status:** active

## AgDR-006 – Local Data Retention on Logout & Trial Abuse Strategy
- **Date:** 2026-07-20
- **Context:** Deciding whether to wipe or reset local macro databases (`macros.db`) and user settings (`config.json`) upon logging out or switching to a new account (e.g. to abuse the 14-day trial).
- **Decision:** Retain all local macro data and configurations globally when users change accounts. Do not wipe or reset any user data on logout.
- **Rationale:** 
  1. Wiping data on logout ruins the user experience for genuine users who change their email or make typos, and risks data loss.
  2. Smart abusers can bypass local wipes easily by copying the AppData files.
  3. Trial-cycling users during the growth phase provide positive word-of-mouth marketing and help popularize the app.
- **Consequences:** Abusers can cycle trials to keep using the app, but they act as free marketing. If abuse severely impacts revenue in the future, we will implement disposable email filters or Hardware ID (CPU/Motherboard) locks in Supabase rather than wiping local client data.
- **Status:** active

## AgDR-007 – Pre-Packaging Website & Master Guide Audit Protocol
- **Date:** 2026-07-21
- **Context:** Ensuring marketing site, pricing structure, and documentation accurately match app capabilities prior to final release packaging.
- **Decision:** Establish a compulsory pre-packaging website audit checklist:
  1. **Master Guide Page**: Standardized to "Master Guide". Complete content audit to add all newly shipped features and remove legacy/outdated instructions.
  2. **Entire Website Scan**: Audit all site pages (`index.html`, `pricing.html`, `faq.html`, `about.html`, etc.) for feature parity, pricing alignment (14-Day Free Trial + $4.99 Lifetime Access), and screenshot accuracy.
  3. **Agent Role**: AI agents must prompt and remind the user to approve the full audit checklist before executing packaging steps.
- **Rationale:** Prevents documentation drift, misleading claims, or broken navigation on launch.
- **Status:** active

## AgDR-008 – Generous AI Quotas & Dynamic Backend Control Strategy
- **Date:** 2026-08-01
- **Context:** Setting AI request limits (Free Trial vs Paid) during initial app launch vs long-term scaling.
- **Decision:** 
  1. **Phase 1 (Launch & Growth)**: Provide generous daily AI quotas to maximize user delight, viral growth, and word-of-mouth:
     - **Free Trial Users**: 20–25 AI requests / day.
     - **Paid / Lifetime Users**: 75–100 AI requests / day.
  2. **Fair Usage Weighting**: 1 Chat Message = 1 request count; 1 AI Builder Macro Generation = 2 requests count.
  3. **Backend Control Knob**: Enforce quotas server-side via Supabase Edge Function (tracked per account ID, not IP) to prevent VPN resets while keeping client code decoupled.
  4. **Phase 2 (Milestone Adjustment)**: Adjust backend quotas down (e.g. 10 Free / 30 Paid) upon reaching any of 3 checkpoints:
     - 5–10 paying customers (revenue covers official paid API keys).
     - 1,000 total app downloads.
     - Monthly server AI API bill reaches $20–$30.
- **Rationale:** Frictionless early usage accelerates growth. Backend enforcement means limits can be adjusted instantly without client app updates.
- **Status:** active

