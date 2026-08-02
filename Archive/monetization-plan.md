---
tags: [status, business, monetization, licensing, launch]
date: 2026-05-24
status: planned
---

# Monetization & Launch Plan 💰

## Launch Strategy (Two-Phase)

### Phase 1: Free + Coffee ☕
- App is **100% free** with a "Buy Me a Coffee" donation button
- No sign up, no licensing, no restrictions
- Goal: get users, get feedback, prove demand
- Track downloads and usage to measure traction

### Phase 2: Licensed Version 🔑
- If Phase 1 is successful → push update with licensing
- Same app, but with sign up + trial + Pro features
- Coffee version stays available as a separate build for comparison

### Transition & Early Adopter Reward Strategy 🎁
To reward early users without requiring a signup system in Phase 1:
- **Master Promo Code (First Come, First Served):** Create a single master promo code (e.g., `EARLYBIRD`) configured inside Lemon Squeezy or verified locally.
- **Activation Limits:** Limit the code to exactly **500 activations** (or a set number of users).
- **FOMO & Engagement:** Distribute this code publicly on Discord, social media, or email updates to build urgency, buzz, and drive community engagement.
- **Transition Flow:** When Phase 2 updates are pushed, active community members who use the code are upgraded to Pro for free, while latecomers pay the standard, reasonable pricing.

---

## User Modes

| Mode | What they get | Requires |
|------|--------------|----------|
| **Guest** | Basic macros, recording, editor — free forever | Nothing |
| **Signed Up** | AI features, Remote control | Free account (email/password) |
| **Licensed (Pro)** | Everything unlocked, no limits | Sign up + purchase license |

---

## Trial Model ⏰

- **Duration:** 14 days
- **What's available during trial:** All Pro features unlocked
- **After trial expires:** Pro features lock, basic features stay free forever
- **Always free:** Basic recording, editor, simple macros
- **Locked after trial:** AI features, Remote, advanced blocks (If/Else, Image Search, Call Macro), unlimited macros

---

## Licensing Flow 🔑

```
User's trial expires
        ↓
App shows "Upgrade to Pro" button
        ↓
User clicks → opens website (Lemonsqueezy checkout)
        ↓
User pays ($50 lifetime / $30 yearly / $5 monthly)
        ↓
Lemonsqueezy takes payment + auto-generates license key
        ↓
Key emailed to user
        ↓
User pastes key in app
        ↓
App checks key with Lemonsqueezy API
        ↓
✅ Unlocked forever (or until subscription ends)
```

### Pricing Options

| Plan | Price | Notes |
|------|-------|-------|
| Monthly | $5/month | Cancel anytime |
| Yearly | $30/year | Cheaper per month |
| Lifetime | $50 one-time | **Best Value** — push this one |

### Payment Provider: Lemonsqueezy 🍋

- Auto-generates license keys — no custom key system needed
- Handles payments, taxes, refunds
- **Fee:** 5% + $0.50 per sale (e.g., $50 sale → you keep $47)
- No monthly fees, no setup cost — only pay when you make a sale
- Has API for your app to verify license keys

---

## Preventing Trial Cheating 🔒

### Reinstall Protection (Hardware Fingerprint)
- App generates a **unique ID** from the PC (CPU + motherboard + disk serial)
- Trial start date stored **locally (encrypted)** + on **Supabase server** linked to hardware ID
- Delete & reinstall → same hardware ID → trial still expired ❌
- New PC → different hardware ID → new trial ✅ (that's fine)

### License Sharing Protection
- License key is **tied to hardware ID**
- Same key on another PC → rejected ❌
- Allow 1-2 activations per key (user can deactivate old PC and move to new one)

---

## Offline Support 📴

- **First launch:** Needs internet once → saves encrypted trial start date locally
- **Next 14 days:** Works fully offline using local encrypted file
- **Trial expires:** App locks → user needs internet to purchase anyway
- **Opportunistic sync:** Whenever the app detects internet, it silently syncs with Supabase in the background — no popups, no forcing, no nagging
- **Tamper protection:** If local encrypted file is deleted/corrupted → app asks to connect to internet to re-verify

---

## Auth System (Supabase) 🔐

- **Provider:** Supabase (free tier — 50k users, 500MB database)
- **Sign up options:** Email/password, Google login
- **What Supabase handles:** Auth, user database, license tracking, hardware IDs, trial dates
- **App communicates via:** REST API calls

### First Launch Screen

```
┌─────────────────────────┐
│   Welcome to PowerX Keys │
│                          │
│   [ Sign Up ]            │
│   [ Log In  ]            │
│   [ Continue as Guest ]  │
│                          │
└─────────────────────────┘
```

---

## Analytics & Tracking 📊

Track seamlessly and invisibly:
- **Download count** — counter on website
- **App opens** — tiny ping on launch
- **Daily active users** — anonymous ping once per day
- **Feature usage** — which features are popular
- **Version tracking** — how many on latest version

Free tools: Google Analytics (website), Supabase (app pings)

---

## Crash Reporting 💥

- **Manual send** — NOT automatic. Show popup: "Something went wrong. Send a crash report?" → user decides
- **Deduplicated** — same crash from 10 users = 1 report with 10 hits, not 10 separate reports
- **Stored in** Supabase (or a simple directory on server)
- **Safety Nets & Safeguards:**
  - **Local Crash Queuing:** Save crashes locally if offline or during a hard shutdown, and prompt to send on the next app startup.
  - **PII Filtering:** Do not include any recorded keystroke or clipboard text in logs. Send only code stack traces.
  - **Rate Limiting:** Max 3 reports per day per unique user to protect the free database tier.
  - **Website Spam Protection:** Integrate **Cloudflare Turnstile** on the manual bug report form to block bot submissions securely.
- **Monthly AI fix workflow:**
  1. End of month → sit down, feed all crash reports to Antigravity/CloudCode
  2. AI reads reports → finds the bugs → applies fixes
  3. AI runs stress tests → verifies 100% working
  4. AI shows summary: "Bug was in X area, causing Y crash. Now fixed."
  5. You confirm by checking the old crash vs new fixed version
  6. If good → push auto-update

---

## Quick Start Guide 📖

- Shows on **first launch only** — simple overlay/popup, 3 steps
- Covers both ways to create a macro:
  1. **Record:** Hit Record → do your actions → Stop → Play
  2. **Manual:** Click Add Action → build your macro step by step
- One "Got it!" button to dismiss — never shows again
- Rough idea for now — details to be discussed later

---

## Marketing Strategy 📢

### Free (Do First — No Money Spent) ✅
- **Product Hunt** — free launch page, thousands of eyes in one day
- **Reddit** — post in r/automation, r/macros, r/software
- **YouTube** — 2-3 min demo video
- **Instagram** — short clips/reels showing cool macros
- **Word of mouth** — share with communities, Discord servers, forums
- **Competitor Comparison Page on Website** — "PowerX Keys vs Macro Recorder vs Pulover's vs TinyTask vs JitBit" comparison table. Shows why PowerX Keys is better. Great for SEO — people Google "Macro Recorder alternative" and find you!

### Paid (Later — Only If App Gets Traction) 💸
- **Microsoft Store** — $19 one-time developer fee, good for visibility and trust
- **Own domain** — buy official website if affordable, otherwise use free subdomain for now
- **Referral program** — "Share with a friend, both get 20% off Pro" (future consideration)

---

## Estimated Implementation Effort

| Task | Time |
|------|------|
| Supabase setup (auth + tables) | 1 hour |
| Login/Signup UI (WPF window) | 1-2 days |
| Auth service in app | 1-2 days |
| Feature gating (check user status) | 1 day |
| License validation (Lemonsqueezy API) | 1 day |
| Hardware fingerprint | 0.5 day |
| Trial logic (local + server) | 1 day |
| **Total** | **~1 week** |

---

## Copyright & Trademark 📜

### Copyright (Do Now ✅)
- Your code is **automatically copyrighted** — you own it the moment you write it
- Add `© 2026 [Your Name]. All rights reserved.` in the app's About page
- Add a **EULA** (End User License Agreement) — show on first install, free templates online

### Trademark (Do Later ⚠️)
- **"PowerX Keys"** is currently **not taken** by any software — name is clear ✅
- "PowerX" alone is used by other companies (PowerX Optimizer, PowerX.ai, Synopsys PowerX) but none use "PowerX Keys"
- Register trademark later if app gets popular (~$250-350)
- Check availability: [USPTO](https://tmsearch.uspto.gov), [WIPO](https://branddb.wipo.int)

### Not Needed ❌
- **Patent** — overkill for a macro app, skip it

### Legal Docs Per Version

| Document | Coffee (Free) | Licensed |
|----------|--------------|----------|
| © Copyright notice | ✅ One line in About page | ✅ Same |
| EULA (first install popup) | ✅ Simple "use at own risk" | ✅ Full version |
| Privacy Policy | ❌ Skip | ✅ Needed (accounts + payments) |
| Terms & Conditions | ❌ Skip | ✅ Needed (accounts + payments) |

### Data & Privacy Disclosure
- App only collects **anonymous usage stats** (daily active user count) — no personal data
- Add this line to EULA: *"This app sends anonymous usage statistics (no personal data) to improve the product. No passwords, no files, no personal information is ever collected."*

---

## Launch Roadmap 🗺️

### Phase 0: Pre-Launch (NOW)
1. Complete pending features
2. Manual testing — check every feature yourself
3. AI agent scan — let agents find remaining bugs/conflicts
4. Make everything bulletproof

### Phase 1: Coffee Launch ☕
5. Launch free version with "Buy Me a Coffee" button
6. Free marketing — Product Hunt, Reddit, YouTube, Instagram
7. Track downloads + daily users

### Phase 2: Licensed Version 🔑 (Side by Side)
8. While free version is live → prepare licensed version with sign up + trial
9. **If good response** → push paid version quickly 🚀
10. **If low response** → build popularity first → then slowly introduce licensing

---

## Source Code Strategy 🔒

- **GitHub repo: PRIVATE** — closed source
- **Why:** Open source = anyone can copy, rename, and sell your app. Can't monetize properly.
- Code stays private, GitHub is only for version control and backup

### Plugin System (Future — v7-v8+)
- Main app stays **100% closed source** 🔒
- Publish a **small open-source API/SDK** — rules for how plugins connect
- Community builds add-ons (macro packs, custom blocks, themes)
- Core app code stays private
- ⚠️ **Security risk:** Plugins can be a backdoor — every plugin must be reviewed before listing
- **Not needed now.** Export/Import macros is enough for launch. Plugins come way later.

---

## Related Pages

- [[known-issues]]
- [[planned-features]]
- [[current-version]]
