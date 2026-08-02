---
tags: [feature, monetization, migration]
date: 2026-07-15
status: pending
---

# Subscription Migration — What to Remove/Replace

**Purpose:** This document lists everything that needs to be removed or replaced when switching from the current free + donation model to a subscription model. The agent building the subscription system should use this as a checklist.

---

## App (Desktop — PowerX Keys V2)

### 1. Tip Jar Popup (MainWindow)
- **File:** `MainWindow.xaml` (lines ~839–900)
- **What:** Bottom-right corner popup that appears after X launches, says "You've executed a lot of macros! This tool is 100% free..."
- **Action:** REMOVE entire popup XAML block (`TipJarPopup` Border)
- **Code-behind:** `MainWindow.xaml.cs`
  - `CheckTipJarStatus()` method — REMOVE
  - `DismissTipJarPopup()` method — REMOVE
  - `ShowTipJar_Click()` method — REMOVE
  - `MaybeLater_Click()` method — REMOVE
  - `OpenTipJar_Click()` method — REMOVE
  - Call to `CheckTipJarStatus()` in constructor — REMOVE

### 2. Tip Jar Section (Settings Dashboard)
- **File:** `Views/SettingsDashboardView.xaml` (lines ~877–890)
- **What:** Card with developer story text + "Support the Developer" button
- **Text to remove:** "I build PowerX Keys entirely in my free time, and it will always remain 100% free..."
- **Action:** REMOVE entire `TipJarContainer` ContentControl
- **Code-behind:** `Views/SettingsDashboardView.xaml.cs`
  - `SupportDeveloper_Click()` method — REMOVE
  - `OnScrollToTipJarRequested()` method — REMOVE
  - Event subscription `ScrollToTipJarRequested` — REMOVE

### 3. Tip Jar ViewModel Properties
- **File:** `ViewModels/SettingsDashboardViewModel.cs`
  - `ScrollToTipJarRequested` static event — REMOVE
  - `TriggerTipJarScroll()` method — REMOVE

### 4. MainViewModel Notification Badge
- **File:** `ViewModels/MainViewModel.cs`
  - `HasUnseenTipJarNotification` property — REMOVE
  - Initialization logic that sets it — REMOVE

### 5. AppConfig Tip Jar Properties
- **File:** `Models/AppConfig.cs` (lines ~144–149)
- **Properties to remove/deprecate:**
  - `TotalMacrosExecuted` — REMOVE (only used for tip jar trigger)
  - `TipPopupShownCount` — REMOVE
  - `HasTipped` — REMOVE
  - `HasDismissedTipPopup` — REMOVE
- **Keep:** `AppLaunchCount` (may be useful for analytics)

### 6. Reset Settings Checkbox
- **File:** `Views/SettingsDashboardView.xaml` (line ~1850)
- **Text:** "Clear All Stats & Timers" mentions "tip jar triggers"
- **Action:** UPDATE text to remove tip jar reference

---

## Website (PowerXKeys-Site)

**Path:** `C:\Users\Maaz\Documents\PowerXKeys-Site\PowerXKeys-Site-main\PowerXKeys_Website\`

### 7. About Page — Developer Story
- **File:** `about.html`
- **What:** Entire page is the "Behind the Keys" story + donation flow
- **Phrases to remove/rewrite:**
  - "PowerX Keys is completely free, with no trial periods, no hidden paywalls, and no locked Pro features. Every single tool is unlocked for everyone."
  - "If you want to support the project, there's a tip jar below, but that's entirely optional."
  - All text that says "free" in context of no payment
- **Action:** Rewrite the story — keep it authentic but remove sympathy/donation angle. Make it professional ("here's why I built this" without "please donate")

### 8. About Page — Donation Section
- **File:** `about.html`
- **What:** "Buy Me a Coffee?" heading + full UPI/Ko-fi donation flow (amount buttons, QR code, confetti)
- **Action:** REMOVE the entire `#support` / `.donation-inline` section
- **Also remove:**
  - All JS functions: `showDonationFlow`, `showUpiFlow`, `cancelUpi`, `generateQr`, `generateCustomQr`, `showCustomInput`, `tipDone`, `showKofiFlow`, `kofiDone`, `cancelKofi`, `triggerConfetti`
  - All CSS: `.donate-trigger`, `.donation-flow`, `.donation-step`, `.method-buttons`, `.method-btn`, `.amount-grid`, `.amount-btn`, `.custom-input-row`, `.qr-display`, `.qr-box`, `.done-btn`, `.cancel-link`

### 9. About Page — Meta Tags
- **File:** `about.html`
- **Meta description:** "...a free Windows macro automation app. Learn why it was built and how you can support the project."
- **OG title:** "Behind the Keys - PowerX Keys Developer Story & Donations"
- **Action:** REWRITE to remove "free" and "Donations"

### 10. About Page — Schema.org
- **File:** `about.html`
- **JSON-LD:** Contains `"sameAs": ["https://ko-fi.com/maazvfx"]`
- **Action:** REMOVE the Ko-fi link from sameAs

### 11. Index Page — Check for "free" messaging
- **File:** `index.html`
- **Action:** Scan for any "free forever", "100% free", "no subscription" phrases and REMOVE/REPLACE with subscription-appropriate language (e.g., "Start your free trial", "Plans start at...")

---

## What to REPLACE with (Subscription)

These spots will need NEW content after removal:
- Tip Jar popup location → could become "Upgrade to Pro" prompt (after trial)
- Settings Tip Jar section → "Manage Subscription" / "Account" section
- About page donation section → "Pricing" or "Plans" CTA
- Website "free" messaging → trial/plan language

---

## Summary

| # | Location | Type | Action |
|---|----------|------|--------|
| 1 | App: MainWindow popup | Tip Jar | REMOVE |
| 2 | App: Settings card | Tip Jar | REMOVE |
| 3 | App: SettingsDashboardVM | Code | REMOVE |
| 4 | App: MainViewModel | Badge | REMOVE |
| 5 | App: AppConfig | Properties | REMOVE |
| 6 | App: Reset checkbox | Text | UPDATE |
| 7 | Web: about.html story | Text | REWRITE |
| 8 | Web: about.html donation | UI + JS | REMOVE |
| 9 | Web: about.html meta | Meta tags | REWRITE |
| 10 | Web: about.html schema | JSON-LD | UPDATE |
| 11 | Web: index.html | Text | SCAN & UPDATE |

