---
purpose: AI Agent Instructions & Wiki Conventions
project: PowerX Keys
version: 1.1
date: 2026-07-09
---

# 📜 SCHEMA.md — AI Agent Instructions

> See [[index]] for the full directory layout and page catalog.

---

## ⚡ Core Workflow Rules

### 1. SPEC-First Development
> For any feature requiring 3+ tasks, write a SPEC **before** coding.

- Create a spec using `templates/spec-template.md`
- SPEC must define: **Outcomes, Scope (in/out), Constraints, Verification Criteria, Task Breakdown**
- Human MUST approve the spec before implementation begins
- If requirements change mid-build → update SPEC first, then code

### 2. Debug Log Before Guessing (Autonomous)
> If you've tried 3+ fixes and the bug persists, **STOP guessing. Automatically create a temporary debug log — do  ask the user for permission.

1. **Detect the trigger** — if the user says "still not working", "same issue", "no luck" after 2+ fix attempts → activate immediately
2. **Create the debug log** — add a temp `DebugLog()` helper that writes to Desktop (e.g., `hook_debug.log`). Log at critical points: object creation, method entry, condition branches, parameter values
3. **Build & tell the user to test** — just say "Test it now, I'll read the log after"
4. **Read the log automatically** — analyze it and pinpoint the root cause
5. **Fix & clean up** — apply the fix, remove ALL debug code, delete the log file, rebuild

---

## 📝 Wiki Rules & Conventions

### Mandatory
- ❌ **NEVER modify files in `raw/`**
- ✅ **ALWAYS update `wiki/index.md`** after adding/removing pages
- ✅ **ALWAYS append to `wiki/log.md`** after any wiki change
- ✅ **Use `[[wikilinks]]`** for all cross-references between wiki pages
- ✅ **Include YAML frontmatter** on every wiki page

### Page Format

Every wiki page must follow this structure:

```markdown
---
tags: [relevant-tag-1, relevant-tag-2]
date: YYYY-MM-DD
sources:
  - source-file-or-code-path
status: active | draft | completed | deprecated
---

# Page Title

**Summary:** One-to-two sentence description.

## Content Sections
(main documentation content with [[wikilinks]])

## Key Files
- [filename.cs](file:///absolute/path/to/file.cs) — brief description

## Related Pages
- [[Related Page 1]]
- [[Related Page 2]]
```

### Tag Taxonomy

- `architecture` — System design docs
- `service` — Service layer components
- `feature` — User-facing features
- `model` — Data models
- `manager` — Manager components
- `status` — Project status/roadmap
- `guide` — Process documentation
- `ui` — UI/View related
- `ahk` — AutoHotkey related
- `ai` — AI integration related

---

## 🗂️ Source Code Location

```
c:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\
├── PowerX_Keys_V2\    ← Main application
└── PowerX_Updater\    ← Standalone updater
```
