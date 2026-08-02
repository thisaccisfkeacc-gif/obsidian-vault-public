---
name: spec-writing
description: Write a SPEC document before implementing any feature that requires 3 or more tasks. Use this when starting a new feature, a significant refactor, or any change that needs user approval before implementation begins.
tags: [skill, spec, planning, development, workflow]
date: 2026-06-08
status: active
sources:
  - SCHEMA.md
  - templates/spec-template.md
---

# 📋 Skill: SPEC-First Development

For any feature requiring 3+ tasks, write a SPEC **before** coding. This prevents wasted effort, misaligned expectations, and mid-build scope creep.

## When to Write a SPEC

| Scenario | SPEC required? |
|----------|---------------|
| Fixing a single bug | ❌ No — just fix it |
| Adding a simple setting | ❌ No — quick change |
| New macro step type (isolated) | ⚠️ Maybe — if it touches 3+ files |
| New UI feature | ✅ Yes |
| Refactoring a service | ✅ Yes |
| New multi-step workflow | ✅ Yes |
| Anything the user hasn't described precisely | ✅ Yes |

## The SPEC Structure

Every SPEC must define these 5 sections:

### 1. Outcomes
What does "done" look like? Write 2-3 measurable success criteria.

*Example:*
> - Users can right-click a macro and duplicate it
> - Duplicate appears in the same folder, named "[Original] - Copy"
> - Undo works for the duplicate action

### 2. Scope (In / Out)
Be explicit about what's included AND excluded. This prevents scope creep.

*Example:*
> **In scope:** Duplicate macro, place in same folder, undo support
> **Out of scope:** Duplicate folder, batch duplicate, copy across folders

### 3. Constraints
What must not be broken? What rules apply?

*Example:*
> - Must not affect existing macro IDs (SQLite primary keys)
> - Must follow existing MVVM pattern — no logic in code-behind
> - Must compile without errors

### 4. Task Breakdown
List each concrete coding task:

```markdown
- [ ] Add `DuplicateMacroCommand` to `MacroEditorViewModel.Commands.cs`
- [ ] Add `DuplicateMacroAsync()` to `MacroDatabase.cs`
- [ ] Add right-click menu item in `ScriptLibraryView.xaml`
- [ ] Wire undo to `UndoRedoManager`
- [ ] Update wiki page [[macro-editor]]
```

### 5. Verification Criteria
How will you confirm the feature works?

*Example:*
> 1. Right-click a macro → "Duplicate" appears in menu
> 2. Click Duplicate → new macro appears named "[Original] - Copy"
> 3. Press Ctrl+Z → duplicate disappears

## SPEC Approval Workflow

```
1. Write SPEC → present to user
2. User reviews → approves or requests changes
3. Only AFTER approval → begin implementation
4. If requirements change mid-build → update SPEC first, then code
5. If new feature conflicts with existing one → flag it in SPEC before proceeding
```

## SPEC File Location

Save SPECs here: `Obsidian Vault/wiki/status/spec-[feature-name].md`

Use the template from `templates/spec-template.md` if it exists.

## SPEC Frontmatter

```markdown
---
tags: [spec, feature, status]
date: YYYY-MM-DD
feature: [feature name]
status: draft | approved | in-progress | completed | cancelled
---
```

## What Makes a Bad SPEC

- ❌ "Improve the SmartMenu" — no outcomes, no measurable criteria
- ❌ Listing implementation details instead of outcomes
- ❌ Scope that's too large for one coding session (split it)
- ❌ Missing the Out-of-Scope section (everything not listed becomes "in scope")
- ❌ No verification criteria (how do you know when you're done?)

## SPEC vs. AgDR

| SPEC | AgDR (DECISIONS.md) |
|------|---------------------|
| Written before work begins | Written after a decision is made |
| Proposes scope and tasks | Records why a choice was made |
| User must approve | Agent writes it autonomously |
| Can be cancelled | Permanent audit trail |
| Feature-level | Architectural-level |
