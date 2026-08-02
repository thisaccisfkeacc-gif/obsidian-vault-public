# 🛡️ High-Impact Website Security Blueprint

> **Target:** PowerX Keys Website & Web Portal  
> **Rule:** High-value, practical safeguards only. Low-impact clutter removed.

---

## 🔥 Essential Safeguards

### 1. 🔒 Backend Database Protection (Supabase RLS)
* **What:** Enforce Row Level Security (RLS) on all database tables.
* **Why:** Stops users from accessing or modifying anyone else's data via public API keys.

### 2. 🛡️ Server-Side Auth Verification
* **What:** Validate session tokens on the backend before serving sensitive data or pages.
* **Why:** Prevents users from bypassing client-side JavaScript checks.

### 3. 🌐 Edge Security Headers & Cache Protection
* **What:** Enforce `Cache-Control: no-store` on `/private` and `X-Frame-Options: DENY`.
* **Why:** Stops admin screens from being saved in browser cache and blocks clickjacking.

### 4. ⏱️ Rate Limiting on Logins
* **What:** Throttle auth attempts (max 5 per minute per IP).
* **Why:** Completely stops automated brute-force password guessing.
