---
tags: [feature, monetization, planning]
date: 2026-07-18
status: planned
---

# PowerX Keys — Monetization Plan

> This document captures all decisions made during the initial monetization planning session.
> Use this as the source of truth before implementing any subscription or licensing system.

---

## ✅ Chosen Model: Free Trial → Paid

**Decision:** No permanent free tier. Just a time-limited free trial, then paid.

**Why NOT a free tier:**
- A free tier trains users to never pay — they get comfortable and lose urgency.
- Free tiers attract low-quality users who never convert.
- A trial creates urgency: "Pay or lose access." That pressure converts better.

**Why NOT free tier + trial + paid (3 tiers):**
- Too complex for users to understand.
- Splits the funnel unnecessarily.

**Why trial → paid works for this app:**
- Users form a habit in 14 days, then feel the loss when it expires.
- Same model used by Notion, Figma, Cursor — proven to convert.

---

## ⏳ Trial Duration: 14 Days

**Why 14 and not 30:**
- 30 days is too relaxed — users forget about the app, trial ends before they get hooked.
- 14 days is enough time to form a daily habit and see value.

---

## 🎟️ Coupon Code System

Coupon codes extend the free trial. Split into 5 tiers:

| Category | Duration | Who Gets It | Example Code Style |
|----------|----------|-------------|-------------------|
| **Creator / Influencer** | 12 months | YouTubers, streamers promoting the app | Unique per person (e.g. `JOHN12`) |
| **Friend & Family / VIP** | 12 months | Inner circle, personal gifting | Private, never posted publicly |
| **Early Supporter** | 6 months | Beta testers, Discord members, loyal community | `EARLY6`, `BETA6` |
| **Event / Launch Promo** | 3 months | Product Hunt, Reddit posts, limited drops | `LAUNCH3`, `PHUNT3` |
| **General Giveaway** | 1 month | Social media giveaways, public posts | `TRY30`, `FREE30` |

**Important rules:**
- Creator codes must be **unique per person** so you can track who drives the most sign-ups.
- General promo codes should have an **expiry date or max redemption limit** to prevent abuse.
- VIP/family codes must **never be posted publicly**.

---

## 💰 Pricing Strategy

> **Goal shifts by stage:** Stage 1 = user base. Stage 2 = revenue growth. Stage 3 = recurring revenue.

### Stage 1 — Early (NOW)
| Plan | Price |
|------|-------|
| Lifetime only | **$4.99** |

> Impulse-buy price. Goal is volume, not margin. At $4.99 people don't think twice — they just buy.
> More buyers = more reviews, word-of-mouth, beta feedback, and social proof.
> No monthly or yearly yet. Keep it dead simple.

### Stage 2 — Growing (User base building)
| Plan | Price |
|------|-------|
| Lifetime only | **$14.99 – $19.99** |

> Price raised once there's a user base. Early buyers feel smart for getting in early.
> Still lifetime only — keeps the offer clean and simple.
> Exact price decided when transitioning based on traction.

### Stage 3 — Significant User Base
| Plan | Price |
|------|-------|
| Lifetime | **$99.99** (skyrocketed) |
| Yearly | **$29.99/year** |
| Monthly | **$3.99/month** |

> When monthly and yearly launch, lifetime price jumps hard.
> This makes lifetime look like a steal for believers, and monthly/yearly feel reasonable for latecomers.
> Real recurring revenue starts here.

---

## 🤔 Why Not Subscription-First?

The user had a valid instinct here. Macro automation is a **niche utility** — not a daily social app. Users think:

> *"Why would I pay $5/month forever for a tool I set up once?"*

This psychology kills subscription conversion for utility apps. **Lifetime pricing works better for tools** (see: Keyboard Maestro, Macro Recorder, AutoHotkey-based tools).

The strategy: make lifetime expensive enough over time that yearly becomes the attractive middle ground — without ever *forcing* it.

---

## ✅ What Pro Unlocks

- Unlimited macros
- AI Assistant (PowerX Intelligence)
- All capture types (Image, Pixel, UI Element)
- Scheduled triggers
- Script Library
- Priority support
- All future features

**Free Trial:** Full access to everything above for 14 days.
**After trial:** App locks until user activates a license or enters coupon code.

---

## 🔐 License Key & Payment System

**Decision: Use LemonSqueezy (do NOT build custom)**

**Why not build custom:**
- License validation, key generation, fraud prevention, payment processing — all extremely complex to build and maintain.
- Security holes in custom systems are a real risk.

**Why LemonSqueezy:**
- Handles **payments + license key generation + coupon codes** all in one platform.
- Provides a **license key validation API** that can be called from inside the app.
- No upfront cost — just a small % per sale.
- Very popular with indie Windows desktop app developers.
- Handles global VAT/taxes automatically.

**Alternatives considered:**
| Option | Verdict |
|--------|---------|
| LemonSqueezy | ✅ Best for indie apps |
| Gumroad | OK, simpler but less control |
| Paddle | More professional, better for scale |
| Custom system | ❌ Too complex, too risky |

---

## 🗺️ Implementation Roadmap (Future)

> These are NOT implemented yet. This is the plan for a future session.

1. **Set up LemonSqueezy account** — create product, plans, and coupon codes.
2. **Build License Validation Service** in the app — calls LemonSqueezy API on launch to check if license is active.
3. **Build Trial Timer** — track trial start date in `AppConfig`, show trial remaining badge in UI.
4. **Build Trial Expired Screen** — a locked screen that appears after 14 days, with "Activate License" and "Enter Coupon" options.
5. **Build Upgrade Prompt** — replace old Tip Jar popup location with "Upgrade to Pro" prompt.
6. **Update Settings Dashboard** — replace old Support section with "Manage Subscription / Account" section.
7. **Update Website** — replace all "free forever" messaging with trial/plan language (see: `subscription-migration-removal-list.md`).

---

## 📝 Website Replacement Plan (High Level)

> Full details in `subscription-migration-removal-list.md`

- `about.html` donation section → Replace with a **"Pricing / Plans" CTA**
- `index.html` "free forever" phrases → Replace with **"Start your free 14-day trial"**
- Meta tags / OG tags → Update to remove donation/free language
- Developer story → Rewrite to be professional (why it was built), no donation angle

---

## 🔐 Auth System & Sign-in

Two sign-in methods:

| Method | Experience |
|--------|-----------|
| **Google OAuth** | 2 clicks, instant — preferred |
| **Email + OTP** | Slightly more steps, but flexible for non-Gmail users |

**Temp mail policy:**
- ✅ Allowed initially — low friction helps adoption early on
- 🔜 Will be blocked later if abuse becomes a noticeable problem
- Monitoring usage patterns before cracking down — no point fighting a problem that doesn't exist yet

**Why no hardware ID anti-cheat:**
- Hardware IDs break on GPU swaps, OS reinstalls, new PCs
- Legitimate paying users would get locked out → support nightmare
- Too complex to build and maintain in early phases
- Completely discarded in favour of soft friction approach

---

## 🎭 Trial Cheat Strategy — Intentionally Soft

**Philosophy:** Let people cheat a little. Make it slightly inconvenient. Don't be aggressive.

The reasoning:
- A genuine paying customer won't bother cheating just to save $5
- Someone who goes through the effort of making a new email to get 14 more days **loves the app** — they are a future paying customer
- Laziness does the filtering naturally
- Being too aggressive signals distrust to honest users

**How cheating works (intentionally):**
1. Trial ends → user creates a new account with a new email
2. They get a fresh 14-day trial
3. All macros and settings are **reset** on the new account (no automatic carry-over)
4. If they want their old data, they must: **Export → Log out → New account → Import**

**Why reset on new account:**
- Export → Import adds just enough friction to make most people think twice
- Completely effortless cheating = no incentive to ever pay
- Committed loyal users will do it — casual free-loaders won't bother
- Keeps the backend simple with no cross-account data headaches

**Scale-up plan:**
- Start open, watch analytics
- If majority of users are cycling accounts and revenue is low → tighten restrictions
- Possible future step: Gmail-only (no temp mail) or OTP verification cooldown per email domain
