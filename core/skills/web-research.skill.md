---
name: web-research
description: Research external topics on the web, evaluate source quality, and ingest findings into the Obsidian wiki. Use when the user asks you to look something up, when a bug requires understanding an external library, or when adding new knowledge to the wiki from online sources.
tags: [skill, research, web, wiki, ingestion]
date: 2026-06-08
status: active
---

# 🌐 Skill: Web Research & Wiki Ingestion

This skill teaches you how to research efficiently, evaluate what you find, and properly integrate discoveries into the PowerX Keys knowledge base.

## Research Workflow

### Phase 1: Define the Query
Before searching, answer:
- What exactly do I need to know?
- What format do I need the answer in? (code example, concept, comparison?)
- Is this likely in a wiki page already? → Check `wiki/index.md` first

### Phase 2: Search Strategically
- Start broad to understand the landscape
- Narrow down with follow-up searches
- For library/API questions: prioritize official docs > GitHub issues > Stack Overflow
- For bug patterns: prioritize official GitHub issues > Stack Overflow
- For architecture/patterns: prioritize authoritative blogs > Reddit discussions

### Phase 3: Evaluate Sources

| Source Quality | Examples | Trust Level |
|---------------|---------|-------------|
| Official docs | Microsoft Docs, AHK docs | ✅ High |
| Official GitHub repo | Repository README, issues | ✅ High |
| Reputable tech blogs | Auth0, Cloudflare, GitHub blog | ✅ High |
| Stack Overflow (accepted answer) | Highly voted, recent | ⚠️ Verify |
| Random blog posts | Medium, Dev.to | ⚠️ Verify against docs |
| Reddit | r/csharp, r/AutoHotkey | ⚠️ Community opinion |

**For AHK specifically:** Always verify against the [official AHK v2 docs](https://www.autohotkey.com/docs/v2/) — v1 answers are everywhere and will break v2 code.

### Phase 4: Synthesize Findings

Don't paste raw search results. Synthesize:
- What did you learn?
- What's the recommended approach?
- Are there trade-offs?
- Does this affect any existing PowerX Keys code?

### Phase 5: Ingest into Wiki

After researching, update the wiki:

1. If the finding updates existing knowledge → edit the relevant wiki page
2. If it's genuinely new (new library, new pattern discovered) → create a new `wiki/references/` page
3. Always cite: `sources: [URL or "Online research: topic"]` in frontmatter
4. Update `wiki/index.md` if a new page was added
5. Append to `wiki/log.md`

## Research Note Format

When dropping findings into a raw research wiki page:

```markdown
---
tags: [research, topic-name]
date: YYYY-MM-DD
sources:
  - URL or description
status: draft
---

# Research: [Topic]

**Question:** What was I trying to find out?

**Summary:** 2-3 sentence answer.

## Key Findings

- Finding 1 (source: URL)
- Finding 2 (source: URL)

## Applicable to PowerX Keys

- How does this affect our codebase?
- Which file/component is most relevant?

## Open Questions

- [ ] What still needs more research?
```

## Existing Research Pages (Check Before Researching)

The wiki already has raw research pages — check these first:
- [[snippet-research]] — Text Snippets system
- [[autofix-research]] — Auto-Fix system
- [[clipboard-research]] — Clipboard Manager

## When Not to Research

- If the answer is clearly in the wiki → don't search, read the wiki
- If the question is about project-specific behavior → read the source code
- If you already know the answer from MEMORY.md → trust your memory

## AHK v2 Research Rule

Always qualify AHK searches with "AutoHotkey v2" or "AHK v2". The vast majority of online content is for v1, which has fundamentally different syntax. Unqualified results will mislead you.
