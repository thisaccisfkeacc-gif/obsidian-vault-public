# T-Drive Full Codebase Audit Report (Revised)

**Generated:** Fresh scan with improved model
**Project:** T-Drive (Telegram MTProto Cloud Storage)
**Files scanned:** server.js (932 lines), style.css (1015 lines), app.js (752 lines), index.html (266 lines), package.json

---

## CRITICAL FINDINGS

### C-01: Real Telegram Credentials Committed in .env
**File:** `.env`
Telegram API_ID, API_HASH, CHANNEL_LINK, and APP_PASSWORD are in plain text. These must be moved to environment variables or a .env file that is `.gitignore`d.

### C-02: `invalidateMessageCache()` Defined But NEVER Called
**File:** `server.js:74-77`
**Severity:** Critical — cache never invalidates after mutations

The function `invalidateMessageCache()` is defined at line 74 but search confirms it is **never invoked** anywhere in the codebase. After any upload (line 698), delete (line 876), or rename (line 909), the message cache remains stale for the full 30-second TTL. Users will not see newly uploaded files, deleted files will still appear, and renamed files will show old names until the cache expires.

**Fix:** Call `invalidateMessageCache()` at the end of `/api/upload`, `/api/files/:targetId` (DELETE), and `/api/files/:messageId/rename` routes.

### C-03: Missing `.red` CSS Class — Disconnected Status Shows Green Dot
**File:** `app.js:121`, `style.css` (class missing)
**Severity:** Critical — false positive status indicator

The frontend injects `<span class="status-dot red">` when Telegram disconnects (app.js line 121). But **no `.red` CSS class exists** in style.css. The `.status-dot` base rule (line 264) sets `background: var(--accent-green)`, so the disconnected indicator renders as GREEN, falsely suggesting everything is fine.

**Fix:** Add `.red { background: var(--accent-red); box-shadow: 0 0 8px var(--accent-red); }` to style.css.

---

## HIGH SEVERITY

### H-01: XSS via `file.name` in `renderFiles()`
**File:** `app.js:302`
`file.name` is inserted raw into both `title="${file.name}"` and `${file.name}`. Executes arbitrary HTML/JS.

### H-02: XSS via `showToast()` innerHTML
**File:** `app.js:702`
`message` parameter inserted raw via `innerHTML`. If message contains user-controlled data (e.g., filenames from Telegram), it executes.

### H-03: Brute-Force Rate Limiter Memory Leak
**File:** `server.js:30, 240-276`
`loginRateLimit` Map grows unbounded. IPs that never hit the limit are never cleaned up. Over time this consumes memory.

### H-04: Share Token Store Memory Leak
**File:** `server.js:72, 451-471`
`publicShareTokens` Map grows unbounded. Expired tokens are only deleted on access. No periodic cleanup.

### H-05: Heartbeat Auto-Shutdown Can Kill Active Uploads
**File:** `server.js:128-147`
If the browser tab is closed during a long upload (>15s gap without heartbeat), `process.exit(0)` kills the server mid-upload. The upload never completes on Telegram.

### H-06: `downloadFile()` Navigates the Page
**File:** `app.js:606-609`
`window.location.href = '/api/download/${messageId}'` causes a full page navigation. While Content-Disposition: attachment keeps most browsers on the page, this creates a history entry, risks losing SPA state, and the heartbeat might fire during download and trigger shutdown.

### H-07: WebDAV Filename Matching Uses Substring
**File:** `server.js:412`
`targetMsg.message.includes(reqPath)` — if reqPath is "re", it matches "report.pdf", "readme.txt", "release.zip". Should use exact match against the filename attribute.

---

## MEDIUM SEVERITY

### M-01: No CSRF Protection on State-Changing Endpoints
All POST/PUT/DELETE endpoints lack CSRF tokens. `sameSite: 'lax'` mitigates but doesn't fully protect.

### M-02: Upload Route Doesn't Call Cache Invalidation (see C-02)
### M-03: Delete Route Doesn't Call Cache Invalidation (see C-02)
### M-04: Rename Route Doesn't Call Cache Invalidation (see C-02)

### M-05: Unclosed `@media` Query — Command Palette Broken on Desktop
**File:** `style.css:903-996`
The `@media (max-width: 768px)` block opens at line 903 but **never closes**. All command palette styles (`.palette-card`, `.palette-input-wrapper`, etc.) are accidentally nested inside the media query and only apply on screens <768px. On desktop, Ctrl+K opens an unstyled modal.

### M-06: No `.spinner` Animation Defined
**File:** `index.html:138`, `style.css` (missing)
Upload icon uses `class="spinner"` but no `.spinner` CSS rule exists. Icon never animates.

### M-07: Upload Progress Bar CSS Conflicts with JS
**File:** `style.css:446-450`, `app.js:572-577`
CSS sets `width: 100%; animation: pulse 1.5s infinite;`. JS sets width dynamically via `style.width`. The pulse animation (opacity oscillation) runs continuously even at 0% width. After completion, JS sets width to 100% but pulse keeps running.

### M-08: Duplicate `.topbar` and `.search-box` Rules
**File:** `style.css:204-212, 484-493` and `214-218, 495-500`
Two definitions each for `.topbar` and `.search-box`. The first blocks (lines 204-218) are dead code — second blocks fully override them.

### M-09: Duplicate `.preview-header` and `.preview-body` Rules
**File:** `style.css:698-709, 761-778`
Both have two definitions that merge unpredictably. `.preview-body` gets `overflow: hidden` from the first and `min-height: 200px` from the second, potentially clipping content.

### M-10: Expired Share Tokens Not Cleaned Up
**File:** `server.js:474-516`
When an expired token is accessed, it's deleted. But tokens that expire without being accessed remain in memory forever. Add a periodic cleanup interval.

---

## LOW SEVERITY

### L-01: `switchTab()` References Non-Existent `#pageTitle`
**File:** `app.js:427-440`
`document.getElementById('pageTitle')` always returns null. Guarded with `if (titleEl)` so no crash, but the dead code is confusing.

### L-02: `closeCommandPalette()` Accepts Unused Event Parameter
**File:** `index.html:243`, `app.js:728-731`
HTML passes `event`, JS function ignores it.

### L-03: `fetchCachedMessages()` Redundantly Calls `resolveChannel()`
**File:** `server.js:79-89`
Both `fetchCachedMessages()` and each route handler call `resolveChannel()`. Either is sufficient.

### L-04: No `.file-meta` Base Definition for Grid View
**File:** `style.css:917-919` (list only)
`.file-meta` is only styled inside `.file-list` context. Grid view has no base rule — spacing works by accident via parent flexbox.

### L-05: `file-list` Override Fragility
**File:** `style.css:882-919`
List view overrides `.file-card`, `.file-thumbnail`, `.file-details` with specific values. Any future CSS change to base classes needs corresponding list overrides.

### L-06: `loginRateLimit` Map Cleanup Never Runs
Non-offending IPs stay in the Map forever. Add periodic cleanup of entries older than 15 minutes.

### L-07: `.env` Contains Hardcoded APP_PASSWORD
**File:** `.env:4`
`APP_PASSWORD=admin123` — default password never changed warning

### L-08: No Stale-While-Revalidate Cache Pattern
**File:** `server.js:79-89`
If cache is refreshing, concurrent requests pile up on the same `client.getMessages()` call. Fine for low concurrency but could time out under load.

### L-09: Rename Replaces Entire Message Text
**File:** `server.js:916-919`
`editMessage()` replaces the entire message text. For files with captions containing metadata, this could lose information. Currently only captions are used so it's safe, but fragile.

### L-10: No TypeScript or Input Validation Library
All validation is manual. No schema validation for API payloads.

---

## SUPPLEMENTARY — BAN PREVENTION GAPS (S Series)

**Status:** `invokeWithRetry` is defined (line 92-108) but **only used in one place** — `fetchCachedMessages()` on line 85. Every other Telegram API call goes directly without retry or rate-limit handling.

### Routes Missing Retry Logic:
- Line 179: `client.invoke(new Api.messages.CheckChatInvite)`
- Line 190: `client.invoke(new Api.messages.ImportChatInvite)`
- Line 203: `client.getEntity()`
- Line 293: `client.sendCode()`
- Line 318: `client.invoke(new Api.auth.SignIn)`
- Line 331: `client.invoke(new Api.account.GetPassword)`
- Line 332: `client.invoke(new Api.auth.CheckPassword)`
- Line 439: `client.sendMessage()`
- Line 486: `client.getMessages()`
- Line 508-511: `client.iterDownload()`
- Line 529: `client.getMessages()`
- Line 548: `client.iterDownload()`
- Line 714: `client.uploadFile()`
- Line 746: `client.uploadFile()`
- Line 759: `client.sendFile()`
- Line 790: `client.getMessages()`
- Line 834-851: `client.iterDownload()`
- Line 858-865: `client.iterDownload()`
- Line 881: `client.getMessages()`
- Line 897: `client.deleteMessages()`
- Line 916: `client.editMessage()`

All of these are vulnerable to `FLOOD_WAIT_` bans. Only `fetchCachedMessages()` has protection.

### Additional Gaps:
- **No inter-upload delay** (except the 500ms in `uploadBatchFiles` frontend — but backend has no backpressure)
- **No progressive backoff** on non-FLOOD errors
- **No request queuing** — if 20 files are uploaded simultaneously, Telegram sees 20 concurrent requests

---

## OSS COMPARISON SUMMARY

| Feature | T-Drive | Teldrive | Pentaract | DriveGram |
|---|---|---|---|---|
| Encryption | None | AES-256 | End-to-end | None |
| Streaming | HTTP Range | Chunked | Adaptive | Basic |
| WebDAV | Basic | Full | None | None |
| Chunking | >2GB | >2GB | >4GB | None |
| Rate limiting | Partial | Full | Full | None |
| Cache | 30s TTL | Stale-while-revalidate | Redis | None |

---

## Improvement Roadmap

1. **Immediate (fixes):** Call `invalidateMessageCache()`, add `.red` CSS class, close `@media` query, add `.spinner` animation
2. **Security:** Add `escapeHtml()` helper, fix XSS in showToast, add CSRF tokens, rotate .env secrets
3. **Performance:** Add cache invalidation on mutations, periodic share token cleanup, add inter-request delay for Telegram API calls
4. **UX:** Fix `downloadFile()` to use `<a>` tag instead of `location.href`, add proper loading states
5. **Telegram Safety:** Wrap ALL Telegram API calls with `invokeWithRetry()`, add progressive backoff, add request queue with concurrency limit
