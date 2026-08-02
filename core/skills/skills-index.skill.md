---
name: skills-index
description: Master discovery manifest for all agent skills in the PowerX Keys project. Read this FIRST at the start of any session — Level 0 discovery. Only ~600 tokens. Then load only the specific skill(s) you need.
tags: [skill, index, discovery, meta]
date: 2026-06-08
version: 2.0
status: active
---

# 🗂️ Skills Index — Level 0 Discovery

> **For agents:** This is your FIRST read of every session (after SOUL.md and MEMORY.md).
> It tells you what skills exist without loading any of them.
> Load only the skill(s) that match your current task.

---

## 🧠 Always-Active Files (Read Every Session)

These are NOT skills — they must be loaded at session start:

| File | When | What it gives you |
|------|------|------------------|
| `AGENTS.md` | Session start (auto-loaded by most tools) | Project overview, build commands, constraints |
| `SOUL.md` | Session start | Your identity, values, decision framework |
| `MEMORY.md` | Session start + session end | Cross-session context (read start, append end) |
| `CHECKPOINT.md` | Before starting any long task | Check if previous agent left off mid-task |
| `DECISIONS.md` | Before any architectural decision | Why things are the way they are |

---

## 📋 Available Skills — Quick Reference

| Skill File | Load When... |
|------------|-------------|
| `multi-agent-system.skill.md` | Running parallel agents, managing a job board, coordinating multiple workers on one project |
| `obsidian-markdown.skill.md` | Writing/editing `.md` files — wikilinks, callouts, frontmatter, embeds, Obsidian-specific syntax |
| `wiki-maintenance.skill.md` | Updating wiki after code changes, ingesting raw docs, querying knowledge, linting wiki health |
| `token-optimization.skill.md` | Long tasks with many files; context growing large; need to decide what to read efficiently |
| `context-compression.skill.md` | Session is getting very long; need to summarize and free context; updating MEMORY.md end-of-session |
| `bug-fixing.skill.md` | Fixing any bug in PowerX Keys — provides the full read → fix → build → verify → wiki workflow |
| `code-review.skill.md` | Reviewing C#/WPF/AHK code for correctness, safety, thread safety, MVVM violations |
| `ahk-scripting.skill.md` | Working with AutoHotkey v2 code in `ScriptCompilerService.cs` or AHK engine scripts |
| `wpf-patterns.skill.md` | WPF XAML, MVVM bindings, INotifyPropertyChanged, Dispatcher threading, DependencyProperty |
| `spec-writing.skill.md` | Feature requires 3+ tasks; writing a SPEC before coding; getting user approval on scope |
| `web-research.skill.md` | Researching external topics, evaluating sources, ingesting findings into wiki |
| `qa-testing.skill.md` | Running or updating the 70-test QA checklist; structuring test results |
| `release-packaging.skill.md` | Building a release, running obfuscation, packaging, pushing to GitHub for auto-update |

---

## ⚡ Progressive Disclosure Pattern

```
Level 0 (this file, ~600 tokens):
  → Know what skills exist
  → Choose which one(s) apply to your task

Level 1 (full SKILL.md, ~1-3k tokens each):
  → Load only the skill(s) for your task
  → Get exact workflow steps for this project

Level 2 (source files, varies):
  → Read only the specific code/wiki files needed to execute
  → Never read source code without Level 1 context first
```

**Token budget rule:** If your task needs more than 2 skills, reconsider — you may be doing too much in one session.

---

## 🔗 Related Files

- `SOUL.md` → Identity & values
- `MEMORY.md` → Persistent cross-session memory
- `CHECKPOINT.md` → Task state / handoff recovery
- `DECISIONS.md` → Architectural decision records
- `AGENTS.md` → Root operating instructions
- `wiki/index.md` → Master knowledge map
