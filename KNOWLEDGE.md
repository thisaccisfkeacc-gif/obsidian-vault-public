---
type: knowledge_map
status: active
summary: Universal AI entry point and folder map for PowerX Keys workspace.
last_updated: 2026-07-28
---

# 🗺️ PowerX Keys — Master Knowledge Map

Welcome to the **PowerX Keys** Knowledge System. All AI agents (Antigravity, Kiro, Claude, ChatGPT) **must read this file first** before navigating the vault or performing codebase searches.

---

## 📂 Vault & Project Map

| Directory / File | Purpose | Access Priority |
| :--- | :--- | :--- |
| `Obsidian Vault/KNOWLEDGE.md` | Universal entry map & context contract (This File) | 1️⃣ Read First |
| `Obsidian Vault/wiki/index.md` | **Master Technical Wiki Index**: Gateway to all 70+ technical docs, services, & guides | 2️⃣ Read Before Deep Research |
| `Obsidian Vault/core/DECISIONS.md` | **Master Quick-Index Log**: 1-line summary list of all architectural decisions for fast scanning | 3️⃣ Read Before Proposing Architecture Changes |
| `Obsidian Vault/wiki/decisions/` | **Detailed ADR Pages**: Full technical breakdown (Context, Rationale, Consequences) for major decisions | 3️⃣ Read For Full Technical Context |
| `Obsidian Vault/core/GOTCHAS.md` | Critical gotchas & bug prevention list | 4️⃣ Read Before Writing Code |
| `Obsidian Vault/wiki/lessons-learned.md` | Living record of solved bugs and edge cases | 5️⃣ Read During Debugging / Issue Resolution |
| `Obsidian Vault/wiki/log.md` | Daily development logs and session handoffs | 6️⃣ Read For Recent Context |
| `Obsidian Vault/ideas/` | Feature ideas, backlogs, and product designs | 6️⃣ As Needed |
| `Obsidian Vault/prompts/` | Multi-AI factory & collaboration prompts | 7️⃣ As Needed |
| `Obsidian Vault/reports/` | System audit reports and security blueprints | 8️⃣ As Needed |
| `Obsidian Vault/T-Drive/` | T-Drive project audit reports, design guidelines, and UI critiques | 8️⃣ As Needed |
| `graphify-out/` | Root-level AST knowledge graph (`graph.json`) | 9️⃣ Use `graphify query` for Code Relationships |
| `code_map/` | Root-level method breakdown files | 🔟 Targeted File Inspection Only |

---

## 📜 Core Architecture Decisions (ADRs)

Before making or suggesting structural code changes, check `Obsidian Vault/wiki/decisions/`:
* [0001-ahk-v2-white-label-engine.md](file:///c:/Users/Maaz/Documents/New%20folder/Obsidian%20Vault/wiki/decisions/0001-ahk-v2-white-label-engine.md) — JSON Config → `ScriptCompilerService` → `.ahk` → `PowerX_Engine.exe`.
* [0002-sqlite-local-storage.md](file:///c:/Users/Maaz/Documents/New%20folder/Obsidian%20Vault/wiki/decisions/0002-sqlite-local-storage.md) — Local SQLite DB (`macros.db`) for instant offline macro execution.

> ⚠️ **Decision Flagging Rule**: AI agents MAY suggest alternative approaches that contradict an ADR, BUT must explicitly flag it:
> `"⚠️ Note: This proposal goes against Architecture Decision ADR-XXXX (Reason: ...)"`

---

## 🏷️ Note Frontmatter Convention

All compiled markdown notes in `wiki/` should begin with YAML metadata:

```yaml
---
type: decision | lesson | guide | backlog
status: active | stale | archived
summary: One-sentence high-signal description for fast AI scanning.
last_updated: YYYY-MM-DD
---
```
