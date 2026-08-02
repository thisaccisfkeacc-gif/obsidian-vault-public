---
name: wiki-maintenance
description: Maintain the PowerX Keys Obsidian wiki — ingest new raw docs, update pages after code changes, query existing knowledge, and run lint checks. Use when updating documentation after a feature or bug fix, or when ingesting a new reference document.
tags: [skill, wiki, obsidian, documentation]
date: 2026-06-08
status: active
sources:
  - SCHEMA.md
  - wiki/index.md
---

# 📚 Skill: Wiki Maintenance

The PowerX Keys wiki is an AI-maintained knowledge base. Keeping it in sync with the codebase is **not optional** — an outdated wiki is worse than no wiki.

## Workflow 1: After Code Changes (Most Common)

Run this after every feature build or bug fix:

1. **Identify affected wiki pages** — what did you change? Which service/feature/model does it touch?
2. **Open each affected page** — read the current content
3. **Update with new information** — what changed? New parameters? New behavior? New edge cases?
4. **Add new pages** if you created a new service, feature, or model
5. **Update `wiki/index.md`** if you added new pages
6. **Append to `wiki/log.md`** with timestamp + what you changed

## Workflow 2: Ingesting a New Raw Document

When a new file appears in `raw/references/`:

1. **Read the raw file thoroughly** (never modify it)
2. **Identify which wiki pages it affects** (or if a new page is needed)
3. **Create/update relevant wiki pages** with the new knowledge
4. **Add `[[wikilinks]]`** connecting related pages
5. **Update `wiki/index.md`**
6. **Append to `wiki/log.md`**

## Workflow 3: Querying the Wiki

When you need to find something:

1. **Start at `wiki/index.md`** — it's the master map
2. **Follow `[[wikilinks]]`** to navigate to the relevant page
3. **Synthesize from wiki pages** — not from raw source files
4. If wiki is insufficient → read source code → **then update the wiki** with what you learned

## Workflow 4: Lint Check

Run periodically to keep wiki healthy:

1. **Orphan pages** — find pages with no inbound `[[wikilinks]]`
2. **Broken links** — find `[[links]]` to pages that don't exist
3. **Stale status** — find pages marked `status: draft` that are actually done
4. **Missing index entries** — pages in `wiki/` not listed in `index.md`

## Page Template

```markdown
---
tags: [relevant-tag]
date: YYYY-MM-DD
sources:
  - source-file-or-code-path
status: active
---

# Page Title

**Summary:** One-to-two sentence description.

## Main Content
(use [[wikilinks]] for cross-references)

## Key Files
- [filename.cs](file:///absolute/path/to/file.cs) — brief description

## Related Pages
- [[Related Page 1]]
- [[Related Page 2]]
```

## Log Entry Format

```markdown
## YYYY-MM-DD HH:MM — [What changed]
- Updated [[page-name]]: [what was added/changed]
- Added [[new-page]]: [what it covers]
- Reason: [why this change was made]
```

## Rules

- ❌ NEVER modify files in `raw/`
- ✅ ALWAYS update `wiki/index.md` after adding/removing pages
- ✅ ALWAYS append to `wiki/log.md` after any wiki change
- ✅ One concept per page — keep pages focused and atomic
- ✅ Use `[[wikilinks]]` for all cross-references, not file paths
