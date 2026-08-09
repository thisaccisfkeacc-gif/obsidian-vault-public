---
name: sprint-pipeline
description: Orchestrate the full sprint testing pipeline for PowerX Keys — from Bug Bash through Triage, Batch Fix, and Targeted Verification. This is the master workflow that coordinates bug-triage and targeted-verify skills into repeating shrinking rounds until zero bugs remain.
tags: [skill, sprint, pipeline, testing, orchestration, otav]
date: 2026-06-30
status: active
sources:
  - wiki/status/testing-checklist.md
  - QA_Testing/sprints/
  - QA_Testing/verify/
  - QA_Testing/dashboard.md
---

# 🏁 Skill: Sprint Pipeline (Master Orchestrator)

**Summary:** The complete Bug Bash → Triage → Batch Fix → Verify pipeline for PowerX Keys. Each round shrinks the bug list until zero remain. This skill coordinates the [[bug-triage]] and [[targeted-verify]] skills into a repeating cycle.

---

## 🔄 The OTAV Cycle (Observe → Think → Act → Verify)

Every sprint round follows the OTAV pattern adapted for PowerX Keys:

| Phase | Who | What Happens |
|-------|-----|-------------|
| **Observe** | User | Bug Bash — test all features, take quick notes |
| **Think** | Manager | Auto-Triage — classify, group, prioritize bugs |
| **Act** | Worker | Batch Fix — fix bugs group by group, critical first |
| **Verify** | User | Targeted Re-test — verify ONLY the fixed items |

Failures and new bugs feed into Round N+1. Each round gets smaller. Repeat until clean. ✅

---

## Step 1: 🧨 Bug Bash Sprint

**Who:** User
**Time:** 20 minutes, strictly time-boxed

### Rules:
- Test ALL features using `wiki/status/testing-checklist.md` as your master checklist
- **Quick notes only** — one line per bug, no deep analysis
- ❌ Do NOT fix anything during testing
- ❌ Do NOT stop to investigate root causes
- ✅ Just record what broke and move on
- ✅ Cover as much surface area as possible

### Output Format:
```
Bug bash notes (raw):
- Settings page: theme dropdown doesn't save after restart
- Macro editor: cursor jumps to line 1 when saving
- Profile system: switching profiles doesn't reload hotkeys
- Script Library: search bar clears when switching tabs
```

---

## Step 2: 📥 Bug Dump + Auto-Triage

**Who:** Manager Agent

User dumps all raw bug notes to the Manager. The Manager then:

1. **Structures** each bug using the [[bug-triage]] template
2. **Classifies severity:**
   - 🔴 **Critical** — crashes, data loss, blocks usage
   - 🟡 **Important** — wrong behavior, missing functionality
   - 🟢 **Minor** — cosmetic, UI polish, typos
3. **Groups by component** — bugs in the same area go together
4. **Checks for root causes** — flags bugs that might share an underlying cause
5. **Cross-references** `wiki/status/known-issues.md` to skip duplicates

---

## Step 3: 📝 Sprint Spec Creation

**Who:** Manager Agent

Manager creates a sprint spec file at:
```
QA_Testing/sprints/sprint-round-N.md
```

### Sprint Spec Template:

```markdown
# 🏁 Sprint Round N — [Date]

**Total bugs:** X
**Breakdown:** 🔴 A critical | 🟡 B important | 🟢 C minor

---

## Group 1: [Component Name] (🔴 Critical)

🐛 BUG-001 | 🔴 Critical | [Component]
→ Description of the bug
→ Steps to reproduce
→ Component: [ViewModel/Service/View]
→ Affected files: [paths]

🐛 BUG-002 | 🔴 Critical | [Component]
→ ...

---

## Group 2: [Component Name] (🟡 Important)

🐛 BUG-003 | 🟡 Important | [Component]
→ ...

---

## Group 3: [Component Name] (🟢 Minor)

🐛 BUG-004 | 🟢 Minor | [Component]
→ ...

---

## ⚠️ Potential Root-Cause Groups

- BUG-001 + BUG-003 may share a root cause in ConfigManager.Save()
- BUG-005 + BUG-006 both involve ScriptCompilerService hotkey generation

---

## 📊 Round Summary

| Metric | Value |
|--------|-------|
| Total bugs | X |
| Groups | Y |
| Root-cause clusters | Z |
| Estimated fixes | W (fewer than bugs if root-cause grouping works) |
```

---

## Step 4: 🗣️ Interactive Planning

**Who:** Manager Agent + User

Before ANY fixing starts:

1. Manager presents the sprint spec to the user
2. User reviews priorities, grouping, and ordering
3. User can:
   - Reprioritize bugs (e.g., promote a 🟢 to 🟡)
   - Defer bugs to a later round
   - Add context the Manager missed
4. User approves → Worker begins

❌ Worker NEVER starts without user approval.

---

## Step 5: 🔧 Batch Fix

**Who:** Worker Agent

Worker reads the approved sprint spec and fixes bugs:

1. **Critical first** (🔴) → then Important (🟡) → then Minor (🟢)
2. Fix **group by group** — all bugs in one component together
3. Root-cause groups are fixed as a single fix when possible
4. Follow the [[bug-fixing]] skill for each fix
5. **ONE `dotnet build` at the end** (not after every fix)
6. If build fails → fix build errors before moving on

### Worker Rules:
- ✅ Fix ONLY what's in the sprint spec
- ❌ No bonus fixes or refactoring
- ❌ No modifying files outside the sprint scope
- ✅ Log what was changed for each bug

---

## Step 6: ✅ Targeted Verification

**Who:** User (guided by Manager)

Manager generates a verification checklist using the [[targeted-verify]] skill:

```markdown
✅ VERIFY ROUND N — Re-test these items:

- [ ] BUG-001: Open settings, change theme, restart app → theme should persist
- [ ] BUG-002: Open macro editor, edit line 5, save → cursor should stay on line 5
- [ ] BUG-003: Switch profiles → hotkeys should reload immediately
```

### User marks each item:
- ✅ **Fixed** — bug is gone
- ❌ **Still broken** — bug persists
- 🆕 **New bug found** — something new broke

---

## Step 7: 🔁 Shrinking Rounds

After verification:

1. ❌ **Still broken** items go to Round N+1
2. 🆕 **New bugs** go through triage and into Round N+1
3. ✅ **Fixed** items are marked done in the checklist and dashboard
4. Each round should be **smaller** than the previous one
5. Repeat until Round N has **zero failures**

```
Round 1: 12 bugs → 9 fixed, 2 still broken, 1 new
Round 2: 3 bugs  → 3 fixed, 0 still broken, 0 new
Round 3: 0 bugs  → 🎉 DONE
```

---

## 📁 File Paths

| File | Location | Purpose |
|------|----------|---------|
| Sprint specs | `QA_Testing/sprints/sprint-round-N.md` | Bug list for each round |
| Verification checklists | `QA_Testing/verify/verify-round-N.md` | Re-test checklist for each round |
| Dashboard | `QA_Testing/dashboard.md` | Overview of all rounds and progress |
| Master checklist | `wiki/status/testing-checklist.md` | The 70-test QA checklist |
| Known issues | `wiki/status/known-issues.md` | Duplicate detection reference |

---

## 🐛 Bug Report Template

```markdown
🐛 BUG-NNN | 🔴/🟡/🟢 Severity | Component Name
→ Description of the bug
→ Steps to reproduce (if known)
→ Component: relevant ViewModel/Service/View
→ Affected files: file paths
```

---

## ✅ Verification Checklist Template

```markdown
✅ VERIFY ROUND N — Re-test these items:

- [ ] BUG-NNN: [exact steps to verify the fix]
  → Expected: [what should happen now]
  → Status: ✅ Fixed / ❌ Still broken / 🆕 New bug
```

---

## 🛡️ Impulse Guard (Preventing Impulsive Implementation)

During testing, the user will often notice small UI tweaks, redesign ideas, or "nice-to-have" improvements. These are NOT bugs — they are **touch-ups** that waste time if implemented during a sprint.

### How the Impulse Guard Works

When the user reports something, the Manager classifies it:

| What User Reports | Classification | Where It Goes | Manager Response |
|-------------------|---------------|---------------|-----------------|
| Crash, data loss, blocks usage | 🔴 Critical | Sprint Spec → Fix NOW | "Critical bug! Adding to sprint." |
| Wrong behavior, missing feature | 🟡 Important | Sprint Spec → Fix this round | "Added to sprint spec." |
| UI tweak, color change, alignment, redesign idea | 🟢 Parking Lot | `QA_Testing/parking-lot.md` | **"Noted! This is a parking lot item. Skip for now, stay focused on the sprint."** |
| Feature idea, "would be nice" | 🟢 Parking Lot | `QA_Testing/parking-lot.md` | **"Good idea! Saved to parking lot. Let's finish testing first."** |

### Manager MUST Actively Push Back

When the user tries to implement a touch-up during a sprint, the Manager should:

1. ❌ **NOT** create implementation plans for touch-ups during a sprint
2. ✅ **Add** the idea to `QA_Testing/parking-lot.md`
3. ✅ **Tell the user:** "This is a parking lot item. I've saved it. Let's stay focused on the sprint — we can do this after testing is done."
4. ✅ **Redirect** the user back to the current sprint phase

### Parking Lot Entry Format

```markdown
| # | Idea | Category | Date Added | From Sprint | Status |
|---|------|----------|------------|-------------|--------|
| 1 | Make settings page dark mode gradient smoother | 🎨 UI Polish | 2026-06-30 | Round 1 | 🅿️ Parked |
| 2 | Add animation to profile switch transition | ✨ UX | 2026-06-30 | Round 1 | 🅿️ Parked |
```

### When to Act on Parked Items

1. ✅ Core sprint is DONE (zero critical/important bugs)
2. ✅ App is stable
3. Then: Go through `QA_Testing/parking-lot.md` and pick what to implement

---

## 📌 Breadcrumb Resume (Never Lose Your Place)

Context switching kills productivity. Every time a session ends or gets interrupted, the agent MUST leave a breadcrumb so the next session picks up instantly.

### When to Write a Breadcrumb
- Before ending ANY work session
- Before switching to a different task
- When the user says "I'll be back later" or similar

### Breadcrumb Format
Write this at the end of the relevant sprint spec or dashboard:

```markdown
📌 RESUME POINT — [Date & Time]
→ Was doing: [what phase of the pipeline / what bug was being fixed]
→ Next step: [the exact next action to take]
→ Open files: [which files were being worked on]
→ Blockers: [anything waiting on user input or external dependency]
```

### Example
```markdown
📌 RESUME POINT — 2026-06-30 14:30
→ Was doing: Round 1 batch fix — Group A (Profile System)
→ Next step: Fix BUG-003 in MacroDatabase.cs line 245, add null check
→ Open files: MacroDatabase.cs, ProfileCreationWindow.xaml.cs
→ Blockers: None — ready to continue
```

### Rules
- ✅ ALWAYS write a breadcrumb before ending a session
- ✅ Keep it to 4 lines max — just enough to resume
- ✅ Be specific — "fix the bug" is useless, "add null check on line 245" is useful
- ❌ Don't write a breadcrumb for trivially short tasks

---

## 🔒 Scope Lock (Prevent Rogue Changes)

Before starting ANY fix or implementation, the agent MUST declare exactly what it will change. This prevents accidental scope creep and makes changes reviewable.

### When to Declare Scope
- Before starting any bug fix
- Before starting any implementation
- Before modifying any file

### Scope Lock Format
```markdown
🔒 SCOPE LOCK — [Task Name]
Files I will modify:
  - filename.cs (line X — what change)
  - filename.xaml.cs (line Y — what change)
Files I will NOT touch:
  - Everything else
```

### Rules
- ✅ ALWAYS declare scope before starting work
- ✅ If you need to touch a file not in scope → STOP and re-declare
- ✅ Keep scope as small as possible
- ❌ NEVER modify files outside declared scope without asking
- ❌ NEVER add "bonus" changes not in scope

---

## 🤖 Decision Auto-Pilot (Don't Ask Trivial Questions)

Too many small questions break the user's focus. The agent should make small decisions independently and only ask about big ones.

### The 5-Minute Rule
> **If reversing this decision takes less than 5 minutes → don't ask, just decide.**

### Don't Ask About (Just Decide)
| Decision Type | Example | Just Do It |
|--------------|---------|-----------|
| Code style | try-catch vs null check | ✅ Pick one and move on |
| Variable names | `isEnabled` vs `enabled` | ✅ Match existing style |
| Error handling approach | Return null vs throw | ✅ Match existing pattern |
| Minor refactoring within scope | Extract to method | ✅ If it makes the fix cleaner |
| Which line to add code | Before vs after existing block | ✅ Use best judgment |

### DO Ask About (Needs User Input)
| Decision Type | Example | Ask First |
|--------------|---------|----------|
| Architecture | New class, new service, new pattern | ❓ Ask |
| Data model changes | Database schema, config format | ❓ Ask |
| Feature behavior | How should this work for the user? | ❓ Ask |
| Breaking changes | Rename public API, change file format | ❓ Ask |
| Scope expansion | "I found 3 more bugs while fixing this" | ❓ Ask |

### Rules
- ✅ Note what you decided in your fix report (transparency)
- ✅ Default to matching existing code patterns
- ❌ Don't ask "should I use X or Y?" for trivial style choices
- ❌ Don't interrupt the user's testing flow with small questions

---

## Related Pages

- [[bug-triage]] — Auto-classification of bugs (severity, component, root cause)
- [[targeted-verify]] — Generating targeted verification checklists
- [[qa-testing]] — The 70-test QA checklist and testing workflow
- [[bug-fixing]] — The verified bug-fix workflow for each individual fix
- [[multi-agent-system]] — Multi-agent coordination (Manager/Worker pattern)
- [[reports/testing-checklist]] — Master QA checklist
- [[parking-lot]] — Ideas Parking Lot for non-urgent touch-ups
