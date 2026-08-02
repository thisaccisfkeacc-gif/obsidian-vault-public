---
name: obsidian-markdown
description: Create and edit Obsidian Flavored Markdown with wikilinks, embeds, callouts, frontmatter properties, and tags. Use when writing or editing .md files in the PowerX Keys Obsidian vault, or when the user mentions wikilinks, callouts, frontmatter, or Obsidian notes.
tags: [skill, obsidian, markdown, wiki]
date: 2026-06-08
status: active
sources:
  - kepano/obsidian-skills (adapted)
  - Obsidian documentation
---

# 📝 Skill: Obsidian Flavored Markdown

Obsidian extends standard Markdown with its own syntax. This skill covers only the Obsidian-specific parts — standard Markdown (headings, bold, lists, tables, code blocks) is assumed knowledge.

## Workflow: Creating a Wiki Page

1. **Check `wiki/index.md`** — does a page for this already exist?
2. **Use the template** from `templates/` if one matches your page type
3. **Add YAML frontmatter** at the top (required for all wiki pages)
4. **Write content** using Obsidian syntax below
5. **Add `[[wikilinks]]`** to connect to related pages
6. **Update `wiki/index.md`** with the new page entry
7. **Append to `wiki/log.md`** with a timestamped entry

## YAML Frontmatter (Required on Every Page)

```markdown
---
tags: [relevant-tag]
date: YYYY-MM-DD
sources:
  - path/to/source/file.cs
status: active | draft | completed | deprecated
---
```

Valid tags: `architecture`, `service`, `feature`, `model`, `manager`, `status`, `guide`, `ui`, `ahk`, `ai`

## Internal Links (Wikilinks)

```markdown
[[Page Name]]                    → Link to another wiki page
[[Page Name|Display Text]]       → Custom display text
[[Page Name#Heading]]            → Link to a specific heading
```

> ✅ Use `[[wikilinks]]` for internal vault links — Obsidian tracks renames automatically.  
> ✅ Use `[text](url)` ONLY for external URLs.

## Callouts (Alerts)

```markdown
> [!note]
> Background context or helpful info

> [!tip]
> Best practice or optimization suggestion

> [!important]
> Essential requirement or must-know

> [!warning]
> Breaking change or potential problem

> [!caution]
> High-risk action — data loss possible
```

## Embeds

```markdown
![[Another Note]]          → Embed full note content
![[Note#Heading]]          → Embed a specific section
![[image.png]]             → Embed an image
```

## Tags

```markdown
---
tags: [service, ai]
---
```

Or inline: `#service` `#ai` (use sparingly — prefer frontmatter tags)

## File Links to Source Code

Always use absolute paths for source file links:

```markdown
[ScriptCompilerService.cs](file:///c:/Users/Maaz/Documents/New folder/PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Services/ScriptCompilerService.cs)
```

## Common Mistakes to Avoid

- ❌ Using `[Page](Page.md)` for internal links — use `[[Page]]` instead
- ❌ Forgetting YAML frontmatter — every page needs it
- ❌ Orphan pages — always add `[[wikilinks]]` from at least one other page
- ❌ Missing `wiki/index.md` update — always add new pages to the master catalog
- ❌ Forgetting `wiki/log.md` entry — always log changes
