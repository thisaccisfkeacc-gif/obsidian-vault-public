---
name: token-optimization
description: Manage context window budget efficiently for long PowerX Keys tasks. Use when you have many files to read, when context is growing large, or when deciding what to read vs. skip. Prevents context bloat, reduces cost, and keeps agents performing at peak quality.
tags: [skill, token, context, optimization, performance]
date: 2026-06-08
status: active
---

# ⚡ Skill: Token Optimization

Context windows are a constrained resource. Treating them carelessly causes performance degradation, forgotten instructions, and failed tasks. This skill teaches you to be deliberately frugal.

## The Core Principle

> **Don't read what you don't need. Read the index, not the encyclopedia.**

Every file you read "costs" tokens that could be used for reasoning and output. The goal is maximum understanding per token spent.

## File Cost Map (PowerX Keys)

Use this to decide what to read and in what order:

| File | Approx Size | Read When |
|------|------------|-----------|
| `skills/skills-index.skill.md` | ~600 tokens | Always — it's your map |
| `SOUL.md` | ~800 tokens | Always — it's your identity |
| `MEMORY.md` | ~1-2k tokens | Always — session context |
| `wiki/index.md` | ~2k tokens | When navigating the project |
| Any wiki page | ~500-1k tokens | When that specific topic is needed |
| `AGENTS.md` | ~1k tokens | Already auto-loaded by most tools |
| `AppConfig.cs` (model) | ~1k tokens | When settings-related |
| `MacroItem.cs` (model) | ~1.5k tokens | When macro step-related |
| `ScriptCompilerService.cs` | **~40k tokens** | ⚠️ Only when AHK generation is the task |
| `MacroEditorViewModel (all 6)` | **~15k tokens total** | ⚠️ Only the specific partial file needed |

## Tiered Reading Strategy

Use this exact order before expanding to source files:

```
1. Read wiki/index.md → find the relevant wiki page name
2. Read that wiki page → understand the component
3. Read ONLY the specific source file for that component
4. If the source file is >5k tokens → read only the relevant function/section
```

**Example (bug in SmartMenu click handler):**
- ❌ Wrong: Read entire `ScriptLibraryViewModel.cs` → `MacroEditorViewModel.cs` → `ScriptCompilerService.cs`
- ✅ Right: Read `wiki/features/smart-menu-ui-ux.md` → find the click handler file → read just that function

## Progressive Disclosure Reminder

```
Level 0 (skills-index): ~600 tokens
Level 1 (one SKILL.md): ~1-3k tokens
Level 2 (one wiki page): ~500-1k tokens
Level 3 (source file section): ~1-5k tokens
TOTAL for a well-scoped task: ~5-10k tokens
```

Contrast with reading everything: **50k+ tokens** — a 5-10x waste.

## Danger Signs (You're Over-Reading)

- You've read more than 3 source code files for a single bug fix
- You've loaded more than 2 full skill files
- You're reading `ScriptCompilerService.cs` for something unrelated to AHK generation
- You're reading model files when the task is UI-only

## When to Skip Reading

You can skip reading a file if:
- The wiki page clearly describes the component's behavior and you're not changing its logic
- You already know the file's structure from MEMORY.md Layer 3
- The task is purely additive (new feature) and doesn't touch existing code

## Context Compression Trigger

When you notice your context is growing very large (many tool calls, many files read), **stop and switch to the `context-compression.skill.md` skill** before continuing.

## Token Budget Guidelines

| Task Type | Target Token Budget |
|-----------|---------------------|
| Wiki update only | < 5k |
| Single bug fix | < 15k |
| Feature with 2-3 changes | < 25k |
| Multi-file refactor | < 40k (use CHECKPOINT.md) |
| Full scan / review | Use multi-agent system — distribute across agents |
