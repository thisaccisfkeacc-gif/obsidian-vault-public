# T-Drive CSS & Frontend Bug Report (Revised)

**Re-scanned with fresh analysis**

## BUG 1 — Unclosed `@media` Query (CRITICAL)
**File:** `style.css:903-996`
`@media (max-width: 768px)` opens at line 903 but the closing `}` is **missing**. All code from line 934 onward (command palette: `.palette-card`, `.palette-input-wrapper`, `.palette-list`, `.palette-item`) is accidentally nested inside the mobile media query. **Ctrl+K palette has zero styling on desktop.**

**Fix:** Insert `}` after `.topbar-actions` rules (~line 932) to close the media query.

---

## BUG 2 — Missing `.red` CSS Class (CRITICAL)
**File:** `app.js:121`, `style.css` (class missing)
`app.js` injects `<span class="status-dot red">` when Telegram disconnects. No `.red` CSS rule exists. `.status-dot` (line 264) always renders `background: var(--accent-green)`. **Disconnected state shows a green dot** — false positive.

**Fix:** Add `.red { background: var(--accent-red); box-shadow: 0 0 8px var(--accent-red); }`

---

## BUG 3 — `invalidateMessageCache()` Defined But Never Called (CRITICAL)
**File:** `server.js:74-77`
Function exists but is **zero-referenced** outside its definition. After upload (line 698), delete (line 876), or rename (line 909), the file list cache stays stale for 30s.

**Fix:** Call `invalidateMessageCache()` at the end of each mutation route.

---

## BUG 4 — Duplicate `.topbar` Definition
**File:** `style.css:204-212` and `484-493`
First block (line 204): no background. Second block (line 484): adds `background: var(--bg-sidebar)`. First block is **dead code**.

---

## BUG 5 — Duplicate `.search-box` Definition (Conflicting)
**File:** `style.css:214-218` and `495-500`
First: `position: relative; max-width: 480px`. Second: `position: absolute; max-width: 440px; flex: 1`. First is **dead code**.

---

## BUG 6 — Duplicate `.preview-header` and `.preview-body`
**File:** `style.css:698-709` and `761-778`
`.preview-body` gets `overflow: hidden` from first rule AND `min-height: 200px` from second — may clip content. Both should be merged.

---

## BUG 7 — Missing `.spinner` Animation
**File:** `index.html:138`, `style.css` (missing)
Upload icon has `class="spinner"` but no `.spinner` CSS exists. Icon never animates.

---

## BUG 8 — Upload Progress Bar CSS/JS Conflict
**File:** `style.css:446-450`, `app.js:572-577`
CSS sets `width: 100%; animation: pulse`. JS overrides width to `0%` then sets it to `percent%`. The `pulse` animation (opacity oscillation) runs at all widths including 0%. After completion, pulse keeps running.

**Fix:** Remove `width: 100%` from CSS. Let JS control width. Use `transition: width 0.3s ease` instead of animation.

---

## BUG 9 — Missing `.file-meta` Base Rule
**File:** `style.css:917-919`
`.file-meta` is only styled inside `.file-list` context. Grid view has no styling for it — spacing works by accident.

---

## BUG 10 — XSS via `file.name` in `renderFiles()`
**File:** `app.js:302`
`title="${file.name}">${file.name}` — raw HTML injection. Filename `<img src=x onerror=alert(1)>` executes.

---

## BUG 11 — XSS via `showToast()` innerHTML
**File:** `app.js:702`
`toast.innerHTML = ... ${message} ...` — raw injection via error messages and filenames.

---

## BUG 12 —— `downloadFile()` Page Navigation
**File:** `app.js:606-609`
`window.location.href` creates history entry and may reload page on some browsers. Better: hidden `<a>` tag with `.click()`.

---

## BUG 13 — `switchTab()` Dead `#pageTitle` Reference
**File:** `app.js:427-440`
`document.getElementById('pageTitle')` always returns null. Guarded but dead code.

---

## Summary

| # | Bug | Severity | Type |
|---|---|---|---|
| 1 | Unclosed `@media` query | Critical | CSS |
| 2 | Missing `.red` CSS class | Critical | CSS |
| 3 | `invalidateMessageCache()` never called | Critical | Backend |
| 4 | Duplicate `.topbar` | Medium | CSS |
| 5 | Duplicate `.search-box` | Medium | CSS |
| 6 | Duplicate preview styles | Medium | CSS |
| 7 | Missing `.spinner` animation | Medium | CSS |
| 8 | Upload bar CSS/JS conflict | Medium | CSS/JS |
| 9 | Missing `.file-meta` base | Low | CSS |
| 10 | XSS `file.name` | High | JS Security |
| 11 | XSS `showToast()` | High | JS Security |
| 12 | `downloadFile()` navigation | Medium | JS UX |
| 13 | Dead `pageTitle` reference | Low | JS |
