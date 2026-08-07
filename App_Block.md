---
type: prd
status: active
summary: Master Product Requirement Document for PowerX App Block (Flutter UI + Native Kotlin Hybrid)
last_updated: 2026-08-06
---

# 🚀 PowerX App Block — Master PRD & Context Document

> ⚠️ **AGENT CONTEXT CONTRACT**: Any AI agent reading this file must read silently, preserve file naming (`App_Block.md`), and execute instructions without dumping raw code or technical jargon into user chat unless requested.

---

## 🎯 Executive Vision
Build a premium, high-performance Android application (APK) featuring a 120fps modern Flutter UI frontend paired with a deep native Kotlin Android backend.

### 💡 Open-Source & Internet Inspiration
* **Existing Reference Codebase:** Native Kotlin reference project located at `C:\Users\Maaz\Projects\StayLocked\Reef\` (`AppBlockManager.kt`, `BlockerService.kt`, `AppBlockScreen.kt`).
* **Design & Community References:** Open-source blocklists, Flutter UI design tokens, awesome-flutter design guidelines, and modern glassmorphism aesthetics.

---

## 📋 Execution & Multi-Agent Strategy

### 1. Pre-Lists & Initial Fast Prototype Phase
* **Initial Build:** Uses pre-configured domain and keyword sample lists for instant prototype generation.
* **Future Scouring Phase:** A dedicated workflow will spawn 10–20 sub-agents in parallel to scour the web, harvest domain lists, and merge them into `master_url_blocklist.json`.

### 2. Two-Prompt Staged Implementation
* **Frontend-First (Prompt 1):** Built using a high-capability model targeting **Live Flutter Web** (`flutter run -d chrome`). Allows instant side-by-side interactive testing and visual approval.
* **Backend Implementation (Prompt 2):** Built after UI approval. Uses **parallel sub-agents** to implement Kotlin system services, method channels, auto-auditing, and false-flag verification.

---

## 🛡️ Core Feature Breakdown

### Phase 1: Adult Content, Keyword Blocker & Master Lock
* **Supported Browser Inspection:** Real-time URL bar extraction via Kotlin `AccessibilityService` (Chrome, Edge, Brave, Firefox, etc.).
* **Fail-Closed Browser Policy:** Any browser where the URL bar cannot be inspected (or secret browsers attempting to bypass filters like Tor) is **instantly blocked from opening**. Shown message: *"Unsupported browser blocked for safety."*
* **Smart Keyword Regex:** Boundary matching (`\bkeyword\b`) to eliminate false flags on safe words (e.g., *"Sussex"*, *"Human Biology"*).
* **Antigravity Master Lock:** Lock status cannot be toggled off or uninstalled without Antigravity key/agent authorization.

### Phase 2: Social Media Time Limits & Reels/Shorts Blocker
* **Time Budgets:** Daily and windowed limits for Instagram, Twitter, Telegram, YouTube.
* **Selective Reels/Shorts Blocker:** Redirects user back to Home Feed when swiping into Reels/Shorts while preserving regular messaging and feeds.

### Phase 3: Bridge & Hybrid Integration
* **MethodChannel / EventChannel:** High-speed IPC between Dart UI and Kotlin backend.
