# T-Drive Fix Prompt — Copy & Forward to Implementation Agent

**Apply all fixes below to:** `c:\Users\Maaz\Documents\New folder\TDrive\`

---

## FIX GROUP 1 — CRITICAL (Do First)

### 1.1 Call `invalidateMessageCache()` After Mutations
**File:** `server.js`

Add `invalidateMessageCache();` at the end of each mutation route:
- After upload success in `/api/upload` (before the `finally` block)
- After delete success in `/api/files/:targetId` (before `res.json`)
- After rename success in `/api/files/:messageId/rename` (before `res.json`)

### 1.2 Add `.red` CSS Class
**File:** `public/style.css` — add after `.status-dot` rule (~line 269):
```css
.status-dot.red {
    background: var(--accent-red);
    box-shadow: 0 0 8px var(--accent-red);
}
```

### 1.3 Close the Unclosed `@media` Query
**File:** `public/style.css` — around line 932, add a closing `}` before `.palette-card`:
- The `@media (max-width: 768px)` block should end after `.topbar-actions`
- All palette styles (`.palette-card` through `.palette-item`) must be OUTSIDE the media query

### 1.4 Add `.spinner` CSS Animation
**File:** `public/style.css` — add after `@keyframes pulse` (~line 456):
```css
.spinner {
    animation: spin 1s linear infinite;
}
@keyframes spin {
    from { transform: rotate(0deg); }
    to { transform: rotate(360deg); }
}
```

---

## FIX GROUP 2 — SECURITY

### 2.1 Add `escapeHtml()` Helper and Fix `renderFiles()`
**File:** `public/app.js` — add near the top:
```javascript
function escapeHtml(str) {
    const div = document.createElement('div');
    div.textContent = str;
    return div.innerHTML;
}
```
Then in `renderFiles()` (~line 302), change:
```javascript
title="${escapeHtml(file.name)}">${escapeHtml(file.name)}
```

### 2.2 Fix `showToast()` XSS
**File:** `public/app.js` (~line 698-705)
Replace `innerHTML` with safe DOM construction:
```javascript
const icon = document.createElement('i');
icon.className = `fa-solid ${type === 'error' ? 'fa-triangle-exclamation' : 'fa-circle-check'} text-accent`;
const span = document.createElement('span');
span.textContent = message;
toast.appendChild(icon);
toast.appendChild(span);
```

---

## FIX GROUP 3 — CSS CLEANUP

### 3.1 Remove Duplicate `.topbar`
**File:** `public/style.css` — delete lines ~204-212 (first `.topbar` block)

### 3.2 Remove Duplicate `.search-box`
**File:** `public/style.css` — delete lines ~214-218 (first `.search-box` block)

### 3.3 Merge Duplicate Preview Styles
**File:** `public/style.css` — replace both `.preview-header` and `.preview-body` definitions with:
```css
.preview-header {
    margin-bottom: 16px;
    padding-bottom: 12px;
    border-bottom: 1px solid var(--border-color);
}
.preview-body {
    display: flex;
    align-items: center;
    justify-content: center;
    min-height: 200px;
    overflow: hidden;
    border-radius: 8px;
    background: var(--bg-main);
}
```

### 3.4 Fix Upload Progress Bar CSS
**File:** `public/style.css:445-450`
Replace `.upload-fill` with:
```css
.upload-fill {
    height: 100%;
    background: linear-gradient(90deg, var(--accent-cyan), var(--accent));
    transition: width 0.3s ease;
}
```
Remove the `pulse` animation from `.upload-fill`.

### 3.5 Add Base `.file-meta` Rule
**File:** `public/style.css` — add before the list override (~line 917):
```css
.file-meta {
    display: flex;
    justify-content: space-between;
    font-size: 0.75rem;
    color: var(--text-muted);
    width: 100%;
}
```

---

## FIX GROUP 4 — JS IMPROVEMENTS

### 4.1 Fix `downloadFile()` to Use `<a>` Tag
**File:** `public/app.js:606-609`
Replace with:
```javascript
function downloadFile(messageId) {
    const a = document.createElement('a');
    a.href = `/api/download/${messageId}`;
    a.download = '';
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
}
```

### 4.2 Remove Dead `pageTitle` Reference
**File:** `public/app.js:427-440`
Remove all `titleEl` references from `switchTab()`.

### 4.3 Fix `closeCommandPalette()` Event Param
**File:** `public/index.html:243`
Change `onclick="closeCommandPalette(event)"` to `onclick="closeCommandPalette()"`

---

## FIX GROUP 5 — BACKEND TELEGRAM SAFETY

### 5.1 Wrap ALL Telegram API Calls with `invokeWithRetry()`
**File:** `server.js`
Identify every `client.xxx()` call and wrap with `invokeWithRetry(() => client.xxx(...))`. Currently only `fetchCachedMessages()` uses it.

Key calls to wrap:
- `client.sendCode()`, `client.invoke()` in OTP flow
- `client.sendMessage()`, `client.sendFile()`
- `client.getMessages()` in download, WebDAV, delete
- `client.uploadFile()`, `client.deleteMessages()`, `client.editMessage()`
- `client.iterDownload()` in streaming routes

### 5.2 Add Periodic Expired Token Cleanup
**File:** `server.js` — add after `publicShareTokens` declaration:
```javascript
setInterval(() => {
    const now = Date.now();
    for (const [token, info] of publicShareTokens.entries()) {
        if (now > info.expireAt) publicShareTokens.delete(token);
    }
}, 60000); // Every 60s
```

### 5.3 Add Login Rate Limiter Cleanup
**File:** `server.js` — add periodic cleanup for stale entries:
```javascript
setInterval(() => {
    const fifteenMinAgo = Date.now() - 15 * 60 * 1000;
    for (const [ip, info] of loginRateLimit.entries()) {
        if (info.lockUntil < fifteenMinAgo && info.count === 0) loginRateLimit.delete(ip);
    }
}, 300000);
```

### 5.4 Fix WebDAV Filename Matching
**File:** `server.js:412`
Replace substring check with exact match against the filename attribute:
```javascript
const targetMsg = messages.find(m => {
    if (!m.media || !m.media.document) return false;
    const nameAttr = m.media.document.attributes.find(a => a.className === 'DocumentAttributeFilename');
    return nameAttr && nameAttr.fileName === reqPath;
});
```

---

## Verification Checklist

After all fixes:
- [ ] Upload a file → appears immediately (cache invalidation works)
- [ ] Delete a file → disappears immediately
- [ ] Rename a file → name updates immediately
- [ ] Disconnect Telegram → status dot turns RED
- [ ] Press Ctrl+K → command palette appears styled on desktop
- [ ] Upload a file → icon spins during upload
- [ ] Upload progress bar → animates smoothly, no pulse after complete
- [ ] Create a file named `<script>alert(1)</script>` → no popup
- [ ] Click download → file downloads without page navigation
- [ ] WebDAV mount → filenames match exactly
