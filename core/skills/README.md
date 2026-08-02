# 🧠 Skills Directory — README

This directory contains the **Procedural Memory** of every AI agent working on PowerX Keys.

Skills are modular, reusable instruction files (`.skill.md`) that teach agents *how to do things* in this specific project — not what the project is (that's `AGENTS.md`) or who the agent is (that's `SOUL.md`).

---

## 🔄 The Progressive Disclosure Pattern

Skills use **3-level lazy loading** to minimize token waste:

```
Level 0 — Discovery (~600 tokens):
  → Read: skills-index.skill.md
  → Result: Know what skills exist without loading any

Level 1 — Activation (~1-3k tokens):
  → Read: The specific SKILL.md matching your task
  → Result: Know exactly how to do this task for this project

Level 2 — Execution (~varies):
  → Read: Only the source files needed to execute
  → Rule: Never read source code without Level 1 context
```

**Start here:** [skills-index.skill.md](./skills-index.skill.md)

---

## 📋 Skill Inventory

| File | Category | Purpose |
|------|---------|---------|
| `skills-index.skill.md` | Meta | Discovery manifest — read first |
| `multi-agent-system.skill.md` | Coordination | Parallel agents, job boards |
| `obsidian-markdown.skill.md` | Documentation | Obsidian syntax |
| `wiki-maintenance.skill.md` | Documentation | Wiki update workflows |
| `token-optimization.skill.md` | Performance | Context budget management |
| `context-compression.skill.md` | Performance | Long-session recovery |
| `bug-fixing.skill.md` | Development | Verified fix workflow |
| `code-review.skill.md` | Development | C#/WPF/AHK code review |
| `ahk-scripting.skill.md` | Specialized | AutoHotkey v2 for PowerX Keys |
| `wpf-patterns.skill.md` | Specialized | WPF MVVM patterns |
| `spec-writing.skill.md` | Process | SPEC-first development |
| `web-research.skill.md` | Research | Research and wiki ingestion |
| `qa-testing.skill.md` | Quality | 70-test QA checklist |
| `release-packaging.skill.md` | Delivery | Build, obfuscate, package, release |

---

## ✍️ Writing a New Skill

Use this format:

```markdown
---
name: skill-name-kebab-case
description: One sentence. When to use this skill. What task it covers.
tags: [skill, relevant-tags]
date: YYYY-MM-DD
status: active
---

# Skill: [Title]

[Brief intro — what problem this solves]

## Workflow / Steps

[Numbered or sectioned content]

## Common Mistakes

[What to avoid]

## Related Pages

- [[wiki-page]]
```

**Rules for good skills:**
- Under 200 lines — if longer, split into two skills
- Lead with a workflow, not theory
- Include project-specific details (file paths, class names, conventions)
- The `description` field is the most important — agents use it for Level 0 discovery

---

## 🔗 System Files (Not Skills)

These live in the vault root, not in `skills/`:

| File | Purpose |
|------|---------|
| `AGENTS.md` | Project overview for all agents (auto-loaded) |
| `SOUL.md` | Agent identity and values |
| `MEMORY.md` | 4-layer persistent session memory |
| `CHECKPOINT.md` | Task state for handoffs and recovery |
| `DECISIONS.md` | Architectural decision records (AgDR) |
| `SCHEMA.md` | Wiki maintenance rules (human-owned) |
