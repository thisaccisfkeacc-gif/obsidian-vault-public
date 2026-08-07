---
type: suggestions
status: active
summary: Side-by-side review and audit suggestions log for App_Block.md
last_updated: 2026-08-07
---

# 💡 PowerX Block — Suggestions & User Testing Feedback

This document stores incoming feedback, testing issues, and UI improvement notes before applying fixes.

---

## 🐛 User Testing Issues Log

### 1. Permission Status Refresh Delay
* **Observed Behavior:** Returning from Android Settings still shows "Grant" until tapped a second time.
* **Fix Plan (For Later):** Auto-refresh on `AppLifecycleState.resumed` so the green tick displays instantly.

### 2. Quiet Website & Keyword Block Behavior
* **Observed Behavior:** When a blocked site or keyword is detected in a browser, `BlockerService` quietly redirects the browser tab (`about:blank`) or kicks the user directly back to the Home screen without showing a custom full-screen overlay.
* **Confirmed:** Working as designed (quiet redirection / kick to Home screen).
