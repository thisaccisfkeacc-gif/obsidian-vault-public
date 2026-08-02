# Multi-AI Brainstorm Spec: AI Usage Limits, Quotas & Lifetime Tiering Strategy

> **Goal**: Solicit feedback, improvements, and strategic ideas from multiple AI models (Kiro, Claude, ChatGPT, DeepSeek, Gemini) to optimize the AI Assistant tiering system in PowerX Keys.

---

## 📌 Context & Current Business Strategy

1. **What PowerX Keys Is**:
   - A **Windows desktop utility** for macro automation: mouse/keyboard actions, image search, pixel detection, window actions, hotkeys, text snippets, and a phone remote control.
   - Audience: automation enthusiasts, gamers, and power users who automate repetitive tasks.
   - It is a **niche utility tool — NOT a mainstream app** (not like Notion/ChatGPT where AI is used constantly all day).

2. **Current Plan Structure**:
   - **Single Simple Paid Tier**: **Lifetime Access** ($4.99). Simple, clean, and easy to maintain for launch.
   - **Future Scope**: Monthly/Yearly subscriptions may be added later, but Phase 1 focuses exclusively on Lifetime Access.
   - **Trial length**: 14 days. Account statuses are strictly **trial / paid / expired** (no other states exist in the DB).
   - **Important**: The AI Assistant is the **ONLY premium-gated feature** — everything else is free. So AI limits are the entire monetization lever; there is no other paywall.

3. **Niche Utility App → Low AI Demand**:
   - Real-world usage pattern: users open the AI a few times per week to build/refine a macro, then go days without touching it.
   - **Implication**: actual average AI usage per user is LOW → even generous daily limits are rarely exhausted → cost per user stays tiny → we can afford to be more generous than a mainstream app would be.

4. **Zero API Cost Baseline (Launch Phase)**:
   - The app currently routes through multiple free AI provider fallbacks/proxies (`AIFallbackService.cs` via Supabase Edge Function).
   - Free trial AI requests cost **$0 out of pocket**, allowing generous limits without financial risk during early launch.

5. **Revenue Is One-Time, Costs Are Recurring**:
   - $4.99 is a **one-time** payment; AI provider costs are **recurring**.
   - This makes the quota design the **primary cost-control lever** — there is no subscription to absorb ongoing AI costs.
   - Expected scale: small niche audience (hundreds, not millions) — no enterprise concerns.

6. **Paid API Key Upgrade Path (10x–20x Boost)**:
   - The primary objective of early sales is to convert 1–2 initial paid lifetime customers.
   - Initial sales revenue will immediately fund official paid API keys (Gemini Pro / Claude / OpenAI).
   - Once official paid API keys are funded, daily limits can easily be boosted **10x to 20x** via backend configuration without requiring client updates.

7. **Dynamic Backend Control**:
   - Daily limits are enforced **server-side** in the Supabase Edge Function (the AI proxy — exact function name to verify).
   - ⚠️ **CURRENT STATE**: quotas are tracked **per IP address** (VPN/WiFi switching can bypass). **PLANNED**: per **User Account ID** tracking via JWT.
   - ⚠️ **CURRENT STATE**: the client sends **NO user identity** in AI requests (only an `X-PowerX-App` header) — the server cannot yet distinguish free vs paid users. Passing the account/tier info from the app is a required prerequisite for per-user quotas.
   - Daily limits can be adjusted dynamically on the backend without pushing app updates to clients.

8. **Current Proposed Quota Rules**:
   - 🆓 **Free Trial Users**: 15 AI requests / day.
   - ⚡ **Lifetime Paid Users**: 50 AI requests / day (permanent, never reduced).
   - ⚖️ **Weighted Deduction**: 1 Chat Message = 1 request count; 1 AI Builder Macro Generation = 2 request counts (due to higher token footprint).
   - 🛡️ **Anti-Abuse Safeguard**: rapid-fire click throttle (max 1 call per 3 seconds) to prevent accidental double-submits.

---

## 🛠️ Codebase & Technical References

- **Client AI Proxy Service**: `PowerX.Services/Services/AIFallbackService.cs`
- **AI Assistant ViewModel**: `PowerX.UI/ViewModels/AIAssistantViewModel.cs`
- **Config & Account Model**: `PowerX.Core/Models/AppConfig.cs` (`IsSubscriptionValid`, `DailyLimit`)
- **Backend Enforcer**: Supabase Edge Function (AI proxy — verify exact function name)

**What the AI Assistant does** (both modes share ONE quota pool):
- **Chat Mode**: conversational Q&A about the app / macros.
- **Build Mode**: generates a complete macro as PowerXMacro JSON, auto-injects it into the editor ("AI Live Build").

```csharp
// Current client-side quota display (the SERVER is the real enforcer; client only reads headers)
// AIFallbackService.cs
public static int RemainingRequests { get; private set; } = 15;  // from "X-RateLimit-Remaining" header
public static int DailyLimit { get; private set; } = 15;          // from "X-RateLimit-Limit" header
```

> ⚠️ Note: the client does **NOT** enforce limits — it only displays them. All enforcement lives in the Edge Function. Don't propose client-side enforcement.

---

## 💭 Current Leanings (Please Challenge These — Don't Just Agree)

*These are our working assumptions. Criticize them freely; we want the strongest possible counterarguments.*

- **Niche-app generosity**: because AI demand is low in a utility app (see Context §3), generous limits are affordable — the numbers below lean generous on purpose.
- **Numbers**: Free 15/day → dial down to 10 later. Paid 50/day (permanent, never reduced).
- **Dial-down checkpoints**: reduce Free limits when reaching ~50 paying customers OR ~1,000 downloads (whichever first) — gradually, never overnight.
- **Zero-state UX**: replace the current hard block at 0 with a **soft grace** — 2–3 bonus requests + a friendly message ("bonus to finish what you started"), plus a warning at ~80% usage ("X requests left").
- **Token economy**: prefer **token-based counting** (builds automatically cost more) over manual weighting (1 build = 2 requests).
- **Anti-abuse**: per-account quota via JWT (currently per-IP — this is a known gap).

---

## 📏 Constraints & Scoring Criteria

Judge every idea against these — simplicity is a feature:

1. **Easy to maintain** — the solo developer must be able to debug it in 10 minutes.
2. **Explainable to users** — no confusing rules ("why is my AI slow?" = bad).
3. **No over-engineering** — avoid queues, adaptive/throttling systems, model-switching logic at this stage.
4. **Safe defaults** — worst case (abuse, cost spike) must not bankrupt the project.

---

## 🎯 Questions & Review Prompts for AI Models

Please review the strategy above and provide structured feedback on the following sections:

### 1. Quota & Value Proposition
- Are **15/day (Free)** and **50/day (Lifetime)** ideal for converting free trial users without overwhelming backend proxies?
- How can we clearly communicate the Lifetime value proposition inside the UI to maximize conversions?

### 2. Fair Usage & Token Economy
- Is the **1 Chat = 1 count, 1 Build = 2 counts** weighting clear and effective, or are there better ways to handle macro builder generation?
- How should the system respond when a user hits 0 remaining requests (e.g. Upgrade Modal, countdown timer to reset at midnight UTC)?

### 3. In-App Conversion Triggers & Nudges
- Should we show an 80% quota warning banner (e.g., *"Only 2 AI requests left today — Unlock 50/day with Lifetime Access!"*)?
- What micro-interactions (e.g., progress bar, animated quota counter) best drive upgrade clicks?

### 4. Transition to Paid API (Scaling Phase)
- Once initial sales fund paid API keys, how should we seamlessly transition from Free Fallback Proxies to Paid Dedicated Endpoints without breaking active client sessions?

### 5. Soft Limits vs Hard Limits
- Critique the "grace requests + warning at 80%" approach. Is it worth the complexity over a clean hard wall?
- What's the single most effective conversion moment when a free user hits zero?

### 6. Cost Control & Checkpoints
- Do the dial-down checkpoints (~50 customers / 1,000 downloads) make sense? What would you change?
- How should limits behave during abuse spikes (someone scripting the AI) without hurting legitimate users?

### 7. Response Format
- Keep your answer structured: **Verdict → Biggest Risks → Top 3 Improvements → Wildcard Idea**. Be specific and concise — no generic advice.

---

## 📝 Agent Feedback & Ideas Log

*(Copy response from each AI model into the sections below)*

### Agent 1 (Kiro)
- *Proposed numbers: Free 15/day, Paid 50/day (permanent).*
- *Per-account JWT enforcement in Supabase Edge Function.*
- *Weighted 1 Chat = 1 count, 1 Build = 2 counts.*
- *Checkpoints: Dial Free to 10 at 50 sales or 1,000 downloads.*
- *Rapid-fire click throttle (max 1 call per 3 seconds).*

### Agent 2 (Claude / ChatGPT / Other)
I read through the full **AI Tiering Strategy** doc. Since it's literally structured as a request for AI model feedback, I'll answer in the exact format you asked for from each agent — here's my take, playing "Agent 2" in your doc.

## Overall Verdict

The core instinct is right — **niche utility app + one-time revenue + AI as the only paywall** means the quota system *is* your entire business model, so it deserves this much thought. Your leanings (generous free tier, JWT over IP, soft grace instead of hard wall, token-weighted builds) are all directionally sound. The main risks are less about the numbers and more about **enforcement gaps you already know about** and a **conversion funnel that's currently invisible to the user**.

## Biggest Risks

1. **Per-IP quota tracking is a real leak, not a theoretical one.** Anyone on mobile data, a VPN, or a shared/dynamic-IP network gets a fresh 15/day automatically. For a $4.99 lifetime price, the incentive to just reset your router beats the incentive to pay. This is your #1 priority — bigger than any number-tuning.
2. **The client sends no identity, so free vs. paid can't even be distinguished server-side yet.** Until `AIFallbackService.cs` passes a signed user/account token, none of the tiering (15 vs 50) can actually be *enforced* — right now it's presumably one shared limit for everyone. This is a blocking prerequisite, not a nice-to-have.
3. **Zero conversion surface.** Nothing in the current spec shows the user *why* they should upgrade at the moment of friction. A hard block (or even a soft-grace block) that doesn't link straight to a purchase flow wastes the single best conversion moment you have.
4. **"Never reduced" for paid lifetime users is a permanent cost commitment with no off-ramp.** If PowerX Keys unexpectedly gets 10,000 downloads instead of hundreds, you can dial Free down, but every Lifetime buyer is locked at 50/day forever on possibly-expensive paid APIs. Worth a stated ceiling clause even if you never plan to invoke it (e.g., "50/day baseline, may temporarily adjust only under extreme abuse").

## Top 3 Improvements

1. **Ship account-based quota enforcement before anything else.** Concretely: have the app send the Supabase JWT (or at minimum a signed account ID + tier claim) with every AI request; have the Edge Function key the rate-limit counter off that ID, falling back to IP only for anonymous/pre-auth requests. This single change closes your biggest hole and makes every other number in the doc actually mean something.
2. **Turn the zero-quota moment into a real upgrade screen, not just a message.** Your "soft grace + 80% warning" idea is good — extend it: at 0 remaining, show a lightweight modal with (a) countdown to reset, (b) the $4.99 lifetime price + what it unlocks (50/day, forever), and (c) a one-click path to checkout. This is the single highest-leverage UI change for revenue, and it costs nothing to build compared to the automation engine itself.
3. **Replace fixed dial-down checkpoints with a simple cost-triggered rule instead of a customer-count rule.** "50 customers OR 1,000 downloads" is a proxy for the thing you actually care about: AI spend. Since you're on free fallback proxies at $0 cost today, downloads/customers don't map cleanly to cost once you move to paid keys. Simpler and safer: track actual daily AI proxy spend (or request volume) in the Edge Function and dial Free down automatically when spend crosses a $ threshold you set (e.g., $20/mo) — this directly ties the lever to the thing that could hurt you financially, and needs no manual checkpoint-watching from you.

## Answers to your specific questions

- **15/Free, 50/Paid** — reasonable starting point given your low-usage-pattern assumption; I wouldn't overthink the exact numbers pre-launch since you can move them server-side. Bigger win: make the *gap* between tiers feel large in the UI (e.g., "3.3x more AI" messaging) rather than agonizing over whether it's 15 vs 12.
- **1 chat = 1, 1 build = 2** — keep this. Token-based counting (your alternative) is more "correct" but adds a debugging dimension a solo dev doesn't need pre-launch. Simple weighted counts satisfy your own "10-minute debuggability" constraint better.
- **80% warning banner** — yes, do it, but make it a passive banner, not a modal (modals should be reserved for the 0-remaining moment so they don't lose impact from overuse).
- **Free fallback → paid API transition** — do it with a server-side feature flag + provider-agnostic interface in the Edge Function only (never in the client), so you can flip providers without any app update — you've already architected this correctly with the `forceUpdate`-style lever pattern you use elsewhere.
- **Soft grace vs. hard wall** — keep soft grace, but cap it hard (e.g., exactly 2 bonus requests, one-time per day) so it can't be gamed into an effectively unlimited tier.
- **Abuse spikes** — the 3-second rapid-fire throttle handles double-submits, but scripted abuse needs a per-account *and* per-IP ceiling stacked together (e.g., "50/day per account, but also max 100/day per IP" to catch multi-account abuse from one machine).

## Wildcard Idea

Turn quota transparency itself into a marketing tool: show a tiny **"Powered by community-funded AI"** note near the quota counter for free users, explaining that paid Lifetime purchases directly fund upgrading everyone (including free trial users) from fallback proxies to faster paid APIs. It reframes the upgrade ask from "pay to remove my limit" to "help fund better AI for the whole PowerX community" — a values-based nudge that costs nothing and fits a solo-dev, indie-app narrative well.

---

### Agent 3 (DeepSeek / Gemini / Other)
I think this is a very solid strategy overall. It's noticeably better than what most indie AI apps launch with because you're optimizing for **simplicity, low maintenance, and cost control** instead of trying to copy ChatGPT's pricing model. The document also clearly explains the business constraints, which helps prevent over-engineering. 

That said, I think there are quite a few areas that could be improved.

---

# Overall Verdict

**8.8/10**

The foundation is excellent.

I would **not** change the single lifetime pricing strategy for launch.

Instead I'd improve:

* quota psychology
* abuse prevention
* conversion UX
* future-proofing
* wording
* backend architecture
* analytics

rather than changing the business model.

---

# Biggest Risks

## 1. Daily limits may actually be too generous

This is probably the biggest thing I'd challenge.

The document assumes users only use AI a few times a week.

That is probably true...

**until AI Builder becomes really good.**

If your macro builder becomes addictive...

People may suddenly create:

* dozens of macro variations
* debugging conversations
* retries
* improvements

instead of only 3-4 requests.

In other words...

**Good AI increases demand.**

Ironically success raises your API bill.

---

## 2. Lifetime pricing locks you forever

The document already acknowledges recurring costs.

But I'd think one step further.

Suppose:

* today GPT-5 Mini costs almost nothing

Then 3 years later

GPT-8

Claude 6

Gemini Ultra

all cost much more.

Your users still own:

> Lifetime

This means your backend must absorb future model pricing.

So I'd architect everything assuming:

> Limits can change.

not

> Models never change.

---

## 3. IP quota should disappear ASAP

I actually wouldn't wait.

I'd prioritize JWT support earlier.

Per-IP quotas are incredibly easy to bypass.

* VPN
* mobile hotspot
* office WiFi
* reboot router

All bypass it.

I think this is your highest priority engineering task. 

---

# Top Improvements

---

# 1. Give users a monthly number too

Instead of only

> 50/day

show

```
50 AI Requests / Day

≈1500 every month
```

People mentally compare

50

vs

15

which doesn't feel huge.

But

```
1500 every month
```

feels enormous.

Same quota.

Higher perceived value.

---

# 2. Show remaining builds instead of requests

Normal users don't understand requests.

They understand tasks.

Instead of

```
18 requests remaining
```

show

```
18 Chats
or
9 Macro Builds
remaining today
```

Much clearer.

---

# 3. Add quota reset countdown

Instead of

```
Resets daily
```

show

```
Resets in

4h 22m
```

This dramatically reduces frustration.

---

# 4. Make Build mode "premium"

Psychologically...

Macro Builder is your killer feature.

Don't advertise

```
50 Requests
```

Advertise

```
AI Macro Builder
Included forever
```

People buy outcomes.

Not quotas.

---

# 5. Remove the wording "Free Trial Users"

This wording makes AI sound temporary.

I'd instead use

```
Trial

15 AI Requests Daily
```

rather than

```
Trial Users
```

Small wording change.

Feels cleaner.

---

# 6. Don't mention numbers until needed

Imagine onboarding.

Instead of

```
15 requests/day
```

show

```
AI Assistant Included
```

Only after several uses show the quota.

Too much emphasis on limits early hurts perceived generosity.

---

# 7. Make upgrade comparison visual

Instead of

```
Trial
15/day

Lifetime
50/day
```

I'd make a tiny comparison card.

```
✓ Chat

✓ AI Builder

15/day

----------------

Lifetime

✓ Chat

✓ AI Builder

50/day

Priority AI

Future AI models

Lifetime updates
```

Even if future items aren't active yet.

It increases perceived value.

---

# 8. Introduce "Power Boost Days"

Fun marketing idea.

Sometimes

Weekend

Holiday

Launch week

Everyone gets

```
100 AI requests today!
```

Costs almost nothing.

Creates goodwill.

---

# 9. Add backend feature flags

Don't hardcode anything.

Instead store:

```
DailyLimit

GraceLimit

BuildMultiplier

CooldownSeconds

WarningPercent

EnabledModels

UpgradeMessage

```

Then every experiment becomes a database edit.

No deployment needed.

---

# 10. Add analytics before changing limits

Never reduce limits blindly.

Track:

```
Average requests/day

Median requests/day

95th percentile

Macro builds/day

Chat/build ratio

Average tokens

Average cost/user

Upgrade after warning %

Upgrade after zero %

```

Without this data you're guessing.

---

# Improvements by Section

---

## Quota

Instead of fixed values forever

store

```
Trial Default

Paid Default

Special Event

Beta Testers

Admin

```

using configurable tiers.

---

## Token Economy

I actually disagree with one point.

The document suggests token-based charging. 

I would **avoid exposing token-based charging to users**.

Users hate invisible pricing.

I'd keep:

```
Chat = 1

Build = 2

```

Simple.

Predictable.

If your backend wants to internally monitor tokens for analytics, that's fine—but don't let token usage determine user-visible deductions.

---

## Warning System

Instead of only

80%

I'd use

```
50%

80%

95%

0%

```

Each message becomes progressively stronger.

---

## Grace Requests

I love this idea.

I'd improve it.

Instead of

```
3 free bonus requests
```

I'd use

```
Need one more build?

We've given you 2 bonus AI requests today.

```

People remember generosity.

---

## Upgrade Modal

Don't interrupt.

Only show after

* successful macro creation

OR

successful AI answer

Never before.

Positive emotion converts better.

---

## Cost Control

Instead of

```
50 customers

1000 downloads

```

I'd also monitor

```
Monthly AI cost

Revenue

```

Something like

```
AI costs exceed

15%

of total revenue

↓

reduce trial

```

Business metrics are more meaningful than download counts alone.

---

# Technical Suggestions

---

### Cache identical prompts

If five users ask

```
How do I create a hotkey?
```

Serve cached responses for common documentation-style questions.

Massive savings.

---

### Retry doesn't consume quota

If

429

500

timeout

network error

provider failure

don't deduct quota.

Users expect to pay only for successful responses.

---

### Deduct after success

Don't deduct before the request.

Only after a valid AI response.

---

### Streaming shouldn't consume twice

If streaming disconnects

resume

don't charge again.

---

### Separate analytics from enforcement

Have

```
Quota Service

Analytics Service

Billing Service

```

Different responsibilities.

Cleaner architecture.

---

### Emergency Kill Switch

One database flag.

```
Disable GPT

Enable Gemini

Reduce Trial

Disable Build

```

Useful if a provider suddenly changes limits or goes down.

---

# Wildcard Ideas

## 1. AI Streak Bonus

Use AI three different days in a week.

Earn

```
+5 bonus requests today
```

Very cheap.

Encourages habitual use.

---

## 2. Referral Boost

Invite a friend.

```
+20 AI requests

```

No subscriptions needed.

---

## 3. Local AI Mode

Eventually support

```
Ollama

LM Studio

Jan

```

Users with local models get unlimited AI.

No API cost.

Huge selling point for power users.

---

## 4. AI Quality Indicator

Instead of model names.

Show

```
⚡ Fast

⭐⭐ Balanced

⭐⭐⭐ Premium

```

Then you can swap providers later without changing the UI.

---

## 5. AI Budget Dashboard (Admin Only)

Create an internal dashboard showing:

* Daily requests
* Token usage
* Cost by provider
* Cost per user
* Cache hit rate
* Upgrade conversion rate
* Error rate
* Provider health
* Average response time
* Abuse detections

This becomes invaluable as your user base grows.

---

# One Thing I Would Definitely Change

The document says that once paid API keys are funded, limits can simply be increased by 10×–20×. 

I would avoid promising or planning a specific multiplier internally. Instead, treat higher limits as something driven by real usage and cost data. If most users only consume a small fraction of their quota, you may discover there's no need to increase limits dramatically—or you may decide to spend the extra budget on better models or faster responses instead. Keeping this decision data-driven gives you much more flexibility in the long run.

---
### Agent 3 
**Verdict**  
The overall strategy is sound for a niche utility with genuinely low AI demand and a one-time $4.99 lifetime price. 15/50 is a reasonable starting point while you are still on free proxies, the weighted count is simple enough, and the planned move to per-account JWT is the single most important fix. The biggest conceptual strength is that you correctly treat AI quotas as the *only* monetization lever and design around real usage patterns instead of pretending this is ChatGPT.

**Biggest Risks**
1. **Per-IP tracking is a silent conversion killer and cost bomb.** Anyone on VPN, mobile hotspot, or shared Wi-Fi can reset the free tier at will. Until JWT + account-based limits ship, the numbers are theater.
2. **One-time revenue vs recurring cost.** $4.99 has to cover years of AI spend for every user who stays active. At current free-proxy rates this is fine; the moment you switch to paid keys the math changes fast if even a small cohort of power users (or abusers) burns 50/day every day.
3. **Soft grace + 80 % banner adds complexity for marginal gain.** Every extra state (grace remaining, warning shown, midnight UTC reset messaging) is another edge case the solo maintainer has to debug at 2 a.m.

**Top 3 Improvements**
1. **Ship per-account enforcement *before* you optimize the numbers.** Make the client send the JWT (or a short-lived signed claim containing tier + user ID) on every AI request. Until that lands, keep free at 10–12 and paid at 40 so the worst-case abuse is smaller. Once it is live, raise the numbers freely via backend config.
2. **Replace the manual 1/2 weighting with a simple token budget that is still displayed as “requests.”** Example: free users get 30 k tokens/day, paid get 100 k. A chat costs ~1–2 k, a full macro build costs 4–8 k. The UI still shows “12 requests left” (calculated server-side from remaining tokens ÷ average cost). Users never see tokens; you automatically charge more for heavy builds without inventing a second currency.
3. **Make the zero-state the cleanest conversion surface, not a soft wall.** When remaining = 0:
   - Hard block (no grace).
   - Single, non-dismissible modal: “You’ve used today’s free AI. Lifetime unlocks 50/day forever — $4.99 once.”
   - Big primary button + secondary “Reset in X hours” (countdown to UTC midnight).
   - Log the event so you can later A/B the copy. Soft grace is nice UX but it dilutes the exact moment the user feels the value gap.

**Wildcard Idea**  
**“AI Day Pass” micro-upsell at the zero wall.** For $0.99 a free/expired user can buy a 24-hour full-paid quota (50 requests or the paid token budget). It is implemented as a one-time Stripe (or whatever) charge that simply writes a temporary higher limit into the user’s row with an expiry timestamp.  
- Gives you incremental revenue from people who are not ready for lifetime.  
- Lets heavy one-day users (macro-building sessions) pay without feeling locked out.  
- Once you have paid API keys the margin is excellent.  
- Keeps the core product simple: still only one permanent paid tier, but you now have a low-friction conversion path that does not require a full $4.99 decision on the first hit of the wall.

Everything else (progress bars, animated counters, 80 % banners) is secondary polish. Nail identity + enforcement + a clean zero-state first; the rest becomes optional.

## ⚡ Shortlisted Action Items (Final Consensus — Logged)

*(Post-noise-filtered consensus across all agents. Final canonical list — log to `Obsidian Vault/ideas/ideas.md`.)*

### 1️⃣ Foundation — Do First
- [ ] Per-account quota via JWT: app sends signed account ID + tier with every AI request; server counts per user (IP fallback only for anonymous calls)
- [ ] All limits as backend feature flags (DB config): daily limits, grace amount, build multiplier, cooldown, warning % — every change = DB edit, no redeploy
- [ ] Stacked ceilings: 50/day per account **plus** max 100/day per IP (catches multi-account abuse from one machine)
- [ ] Charge only on success: retries / timeouts / 429s / provider errors never consume quota
- [ ] Emergency kill switch: one DB flag to disable a provider or cut trial limits

### 2️⃣ Numbers & Economy
- [ ] Keep Free 15/day, Paid 50/day (permanent); keep 1 Chat = 1 count, 1 Build = 2 counts
- [ ] Drop the "10x–20x boost" promise — go data-driven instead
- [ ] Dial-down rule = cost-triggered: reduce Free when AI cost exceeds ~15% of revenue (50-customers / 1,000-downloads as backup triggers)
- [ ] Lifetime ceiling clause: "50/day baseline, may temporarily adjust only under extreme abuse"

### 3️⃣ Zero-Moment Conversion
- [ ] 2 grace requests at zero with warm message ("We've given you 2 bonus requests today"), capped once per day
- [ ] At zero: proper upgrade modal (countdown to reset + $4.99 price + what it unlocks + one-click checkout)
- [ ] Show modal only after a success (never interrupt mid-flow)
- [ ] Progressive warnings at 50% / 80% / 95% as passive banners

### 4️⃣ Quota Psychology (Cheap Wins)
- [ ] Show "18 chats or 9 macro builds remaining" instead of raw requests
- [ ] Frame paid as "≈1500/month"
- [ ] Reset countdown ("Resets in 4h 22m")
- [ ] Advertise "AI Macro Builder — included forever" (sell outcomes, not quotas)
- [ ] Don't show limits during onboarding — reveal after a few uses
- [ ] Visual free-vs-lifetime comparison card at upgrade moment
- [ ] "Powered by community-funded AI" note for free users
- [ ] Wording: "Trial" instead of "Free Trial Users"

### 5️⃣ Small Fairness/Cost Wins
- [ ] Cache common documentation-style questions server-side (e.g., "how do I create a hotkey?")
- [ ] Minimal analytics before dialing anything down: avg/median/95th daily usage, chat:build ratio, upgrade-after-zero %

### 📦 Back Pocket (v2 — Not Now)
- [ ] Track tokens internally on the backend while still displaying "requests" to users (token-accurate cost control without exposing token pricing)
- [ ] AI Day Pass micro-upsell (needs Stripe)
- [ ] Local AI mode (Ollama / LM Studio)
- [ ] Streak bonuses / referral rewards / Power Boost Days
