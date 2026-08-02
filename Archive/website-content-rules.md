---
tags: [guide, website, rules]
date: 2026-06-04
sources:
  - C:\Users\Maaz\Desktop\PowerXKeys-Site\PowerXKeys-Site-main\PowerXKeys_Website
status: active
---

# Website Content Rules 🌐

**Summary:** Mandatory rules for any AI agent working on the PowerX Keys website. These rules ensure the site stays user-friendly, safe, and search-engine optimized.

## 1. ✍️ Writing Style — Plain Language Only

> [!important] **No technical jargon.** Write like you're explaining to a friend who has never coded in their life.

- ❌ Never use developer terms like: "endpoint", "API", "JSON", "SQLite", "database lock", "debouncing", "thread-yielding", "spin waits", "P/Invoke", "metadata-only pings"
- ❌ Never expose internal code names like: `IsDebugHighlight`, `CaptureMousePosition`, `UseAsyncRecording`, `SmartBundling`
- ✅ Use everyday words: "file" instead of "database", "server" instead of "endpoint", "settings" instead of "configuration blocks"
- ✅ When a technical term is unavoidable (e.g. "AutoHotkey"), briefly explain what it means in parentheses
- ✅ Keep sentences short. If a sentence has a comma-separated list of 3+ technical terms, rewrite it
- ✅ Read every sentence and ask: *"Would my non-tech friend understand this?"* — if no, simplify

### Examples

| ❌ Don't Write | ✅ Write Instead |
|---|---|
| "metadata-only pings to our Supabase database" | "anonymous counters to our secure server" |
| "non-destructive post-processing engine" | "macros are automatically cleaned up for easier reading" |
| "connection synthesis tones" | "sound alerts when devices connect" |
| "10 stack levels deep" | "built-in safety limits prevent infinite loops" |
| "three-tier sleep system with thread-yielding" | "smart timing for smooth movement" |

---

## 2. 🛡️ Safety — Never Promote Anti-Cheat Evasion

> [!caution] **Do not add content that could get the app flagged or banned from platforms.**

- ❌ Never use words like: "stealth", "undetectable", "bypass anti-cheat", "evade detection"
- ❌ Never frame features as tools to cheat in games
- ✅ Use neutral language: "Human-Like Input", "natural mouse movement", "realistic delays"
- ✅ If discussing anti-cheat compatibility, be honest about risks (like the FAQ does) — don't promote evasion

---

## 3. 🔍 SEO — Keep Search Engines Happy

- ✅ Every page MUST have: `<title>`, `<meta name="description">`, and a single `<h1>`
- ✅ Every page SHOULD have: Open Graph tags (`og:title`, `og:description`, `og:image`) and `<link rel="canonical">`
- ✅ Use proper heading hierarchy: H1 → H2 → H3 (no skipping levels)
- ✅ Use semantic HTML: `<main>`, `<nav>`, `<footer>`, `<section>`, `<article>`
- ✅ Add `aria-label` and `aria-expanded` to interactive elements (buttons, accordions)
- ✅ Add `aria-hidden="true"` to decorative SVG icons
- ❌ Don't render important content purely via JavaScript if it can be static HTML (search engines can't index JS-rendered text reliably)

---

## 4. 🎨 Tone & Voice

- **Personality:** Confident, honest, slightly casual — like a skilled indie developer, not a corporation
- **Audience:** Regular Windows users, gamers, creators, office workers — NOT developers
- **"We" vs "I":** Use "we" on formal pages (Privacy, FAQ). Use "I" only on the Behind the Keys (personal story) page
- **Humor is OK** but keep it subtle — no memes, no cringe

---

## 5. 📋 Checklist Before Publishing

Before pushing any website change, verify:

- [ ] No technical jargon in user-facing text
- [ ] No anti-cheat evasion language
- [ ] Page has `<title>`, `<meta description>`, and `<h1>`
- [ ] Page has OG meta tags for social sharing
- [ ] All interactive elements have ARIA attributes
- [ ] Footer version number matches current release
- [ ] Links use `.html` extensions for local previewing

---

## Related Pages
- [[agent-onboarding]]
- [[SCHEMA]]
