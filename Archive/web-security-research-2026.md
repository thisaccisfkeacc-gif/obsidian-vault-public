# ⚡ Essential High-Impact Web Security Checklist

> **Goal:** High-value, practical security rules only. Clutter and low-impact tweaks removed.

---

## 🔥 The 4 High-Impact Essentials

### 1. 🔒 Supabase Row Level Security (RLS) & Key Safety
* **Impact:** **CRITICAL**
* **Action:** Enable RLS on all database tables so users can only read/write their own data. Never expose `service_role` keys on the frontend.

### 2. 🛡️ Server-Side Auth Verification
* **Impact:** **HIGH**
* **Action:** Verify user login tokens on the server/backend API before showing private data. Client-side HTML/JS hiding is never trusted alone.

### 3. 🌐 Edge Security Headers & Cache Protection
* **Impact:** **HIGH**
* **Action:** 
  * `Cache-Control: no-store` on private pages so admin screens vanish when closed.
  * `X-Frame-Options: DENY` to block clickjacking attempts.

### 4. ⏱️ Login Rate Limiting
* **Impact:** **HIGH**
* **Action:** Limit login attempts (e.g. max 5/min) to prevent brute-force automated password guessing.
