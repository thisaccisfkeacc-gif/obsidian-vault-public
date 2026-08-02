# 🖥️ Admin Portal (Private Page) - Security Hardening & Roadmap

> **Scope:** This covers the website admin portal (`/private` page) using Dodo Payments.
> For app licensing/monetization strategy, see [[monetization-plan]].

This document outlines the security audit details, future roadmap items, and coupon splitting rules for the Admin Portal (`/private` page) of PowerX Keys.

---

## 🔒 Security Audit & Hardening Guide

The Admin Portal currently uses a single password check to unlock the front-end interface. To make it 100% secure against hackers, implement the following steps:

### 1. 🛡️ API Route Verification (Critical)
* **The Vulnerability**: Currently, `/api/discounts` and `/api/products` endpoints do not check if you are logged in. Anyone who inspects the webpage can copy the API URL and access your Dodo Payment configurations directly.
* **The Fix**: 
  1. Pass the entered admin password in the headers of all API calls (e.g. `X-Admin-Password: your_password`).
  2. In your Vercel serverless function (e.g., `api/discounts/index.js`), compare the header password against your server environment variable before returning any data.

### 2. ⏳ Login Brute-Force Rate Limiting
* **The Vulnerability**: There is no delay or limit on password entry attempts. A script could guess millions of passwords to unlock your dashboard.
* **The Fix**: Add a delay or limit in Supabase RPC or Vercel middleware to block requests from an IP address if they fail the password check 5 times in a row.

### 3. 🔑 Supabase Admin Session Auth
* **The Fix**: Instead of a shared text password, migrate the portal to actual logged-in user accounts via **Supabase Auth**. This ensures only your specific admin email can authorize database transactions.

### 4. 🤖 CAPTCHA Bot Protection (e.g., Cloudflare Turnstile)
* **The Vulnerability**: Automated scripts can brute-force submit the login form continuously to guess the password.
* **The Fix**: Integrate a modern, user-friendly CAPTCHA system (like **Cloudflare Turnstile** or **Google reCAPTCHA v3**) on the login form. The server will reject password guesses unless a valid CAPTCHA token is submitted, blocking automated scripts completely.

---

## 📋 Saved Coupon splitting Rules & Context

Use this reference when you are ready to create your discount codes in the future.

### 📅 1. 30-Day Keys (1,000 Total Uses)
* **Rule**: Split into **10 unguessable codes** of **100 uses** each (using random suffixes like `PXK30-XXXXX`).

### 📅 2. 90-Day Keys (300 Total Uses)
* **Rule**: Split into **10 unguessable codes** of **30 uses** each (e.g., `PXK90-XXXXX`).

### ⭐ 3. 6-Month Keys (Special / Custom-Made)
* **Rule**: Max **20 to 25 unique custom codes** (e.g., `PXK6M-XXXXX`).
* **Limit**: Individual limit of **1** or **2** uses per code, intended for partners/special giveaways.

### 👑 4. 12-Month Keys (Premium / VIP)
* **Rule**: At least **10 unique premium codes** (e.g., `PXK12M-XXXXX`).
* **Limit**: Single-use code (Limit of **1** use per code).

---

## 🚀 Potential Future Features (Roadmap)

### 1. 📅 Expiry Date (Valid Until)
* **What it is**: Automatically disables a coupon code after a specific date and time (e.g., "Ends Sunday at midnight").
* **How to implement**: Add a checkbox toggle `Set Expiry Date` in the UI that reveals a date picker, and pass the selected date to Dodo's API.

### 2. 💲 Discount Type Toggle (Percentage vs. Flat Cash)
* **What it is**: Choose between percentage-off (e.g., `100% OFF`) and a fixed cash discount (e.g., `$10 OFF`).
* **How to implement**: Add a small `%` and `$` selector switch next to the discount amount input field.

### 3. 📊 Advanced Telemetry Expansion
* **OS Distribution**: Track Windows 10 vs. Windows 11 users to optimize compatibility focus.
* **Feature Usage Stats**: Count which macro types, triggers (pixel, image, hotkey), or actions are executed most often to see what features users love.
* **Proactive Crash/Error Tracking**: Monitor and aggregate app errors/crashes so you can fix bugs before users complain.
* **Retention Metrics**: Track Daily Active Users (DAU) vs. Monthly Active Users (MAU) to measure user engagement.

### 4. 💳 Monthly & Annual Subscription Tiers (Planned Expansion)
* **Goal**: Introduce recurring **Monthly ($X/mo)** and **Annual ($Y/yr)** subscription plans alongside current 14-Day Free Trial & Lifetime Access ($29).
* **Supabase Schema Migration**:
  ```sql
  ALTER TABLE public.user_subscriptions 
  ADD COLUMN plan_type TEXT DEFAULT 'lifetime', -- 'monthly', 'annual', 'lifetime'
  ADD COLUMN current_period_ends_at TIMESTAMPTZ;
  ```
* **Dodo Payments & API Updates**:
  1. Register Monthly and Annual recurring product IDs in Dodo Payments Dashboard.
  2. Update `/api/checkout.js` to accept dynamic `product_id` query parameters (`checkout.js?plan=monthly` / `checkout.js?plan=annual`).

---

## 🤖 Security Audit Prompt (Copy/Paste to AI)

Use this prompt when ready to prepare the `/private` page for production:

```markdown
You are an expert security engineer and full-stack developer. Your task is to audit, optimize, and secure the newly created `/private` page and its associated backend API/serverless endpoints on this website.

## Objective
Review the implementation of `private.html` (and any proxy serverless functions/backends that communicate with Dodo Payments) to make it production-ready and bulletproof against compromise.

## Your Workflow
1. **Audit & Scan**: Review the code for security holes, potential key leaks, CORS configuration, and access controls.
2. **Confidence-Based Action**:
   * **Rule 1 (Direct Implementation)**: If you are 100% confident in a standard security improvement or code optimization (like adding rate-limiting headers, sanitizing input fields, or improving error handling), **directly write the code and apply the fix silently**.
   * **Rule 2 (Consultation)**: If an upgrade involves a design choice, changing third-party libraries, or risky architectural changes, **stop, explain your suggestions clearly, and ask for user approval first**.

## Key Focus Areas
- **Supabase Authentication Guard**: Verify that only the admin email (`maazlogomaker@gmail.com`) can successfully load data or call proxy endpoints.
- **Rate Limiting**: Implement basic rate-limiting to prevent automated spamming of coupon code generation.
- **Input Sanitization**: Clean all inputs (such as coupon codes, limits, percentages) on both frontend and backend before sending requests to Dodo Payments.
- **Error Handling**: Mask internal Dodo API details/errors so they are never leaked to the client browser logs.
```
