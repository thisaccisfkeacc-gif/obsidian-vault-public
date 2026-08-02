---
name: multi-agent-system
description: How to coordinate multiple AI agents working in parallel on the same project using a shared file system
tags: [skill, agents, coordination, parallel-work, job-board]
date: 2026-06-07
status: active
version: 2.0
sources:
  - Online research: multi-agent coordination patterns
  - Online research: manager-worker architecture
  - Online research: parallel bug scanning best practices
---

# 🤖 Skill: Multi-Agent Coordination System

**Summary:** How to run multiple AI agents in parallel, each doing their own piece of work, using shared files to communicate — no coding framework required.

---

## 🧠 What This System Is

You have **one Manager** and **many Workers**:

- **Manager** = You (the human) or a designated "boss" agent. Breaks big tasks into small pieces and posts them on the job board.
- **Workers** = Multiple AI agents (each opened in a new chat). They read the job board, grab a job, do the work, and report back.
- **Job Board** = A shared markdown file everyone reads and writes to. This is the heart of the system.

The key insight: **agents don't need to talk to each other**. They just read the same file. Simple and powerful. ✅

---

## 📋 The Job Board File (Core of the System)

The job board is a single `.md` file with this structure:

```markdown
## 📋 JOBS

### Job 1: [Area Name]
**Status:** 🟢 FREE
**Files to scan:** [list of files]
**Focus:** [what to look for]
**Output file:** [where to write findings]

### Job 2: [Area Name]
**Status:** 🔵 TAKEN | [agent-id] | [timestamp]
...
```

### Status Flow:
```
🟢 FREE → 🔵 TAKEN (agent claims it) → ✅ DONE (agent finishes it)
```

---

## ⚡ Improvements From Research (v2.0 Upgrades)

### ✅ Upgrade 1: Add a "Verifier" Role
**Old way:** Worker scans → writes findings → done.
**New way:** Worker scans → writes findings → a **Verifier agent** checks if the findings are real.

This catches false positives and makes results more trustworthy.

```markdown
### Job 1: [Area] — SCAN
**Status:** ✅ DONE | agent-abc | 2026-06-07
**Verifier Status:** 🟢 UNVERIFIED
```

After all scan jobs are done, a second wave of agents picks up UNVERIFIED jobs and confirms the findings.

---

### ✅ Upgrade 2: Smarter Agent Onboarding File

Instead of giving agents a long prompt, create a single **`AGENTS.md`** or **`agent-onboarding.md`** that every agent reads first. It should include:

- What the project is (1 paragraph)
- Where to find the job board
- The exact steps to follow (numbered)
- What NOT to do
- The output format to use

**Rule:** Keep it under 200 lines. Long instructions make agents forget things at the bottom.

```markdown
# Agent Onboarding

1. Read this file top to bottom
2. Go to master-scan-jobs.md
3. Find the first 🟢 FREE job
4. Change its status to 🔵 TAKEN | [your first 8 chars of conversation ID] | [time]
5. Read every file in your job's scope
6. Write findings to the output file path listed in your job
7. Change job status to ✅ DONE
8. Tell the user you're done
```

---

### ✅ Upgrade 3: Specialist Agent Types

Don't give all agents the same job. Research shows **specialized agents do better** than general ones:

| Agent Type | What They Focus On |
|------------|-------------------|
| 🔴 Security Agent | Passwords, injections, path traversal, unencrypted data |
| ⚡ Performance Agent | Slow loops, heavy operations, memory leaks |
| 🧵 Thread Safety Agent | Race conditions, shared state, async void |
| 🧹 Polish Agent | UI bugs, wrong labels, missing feedback |
| ✅ Verifier Agent | Checks if bugs were actually fixed |

You can label jobs by type in the job board:

```markdown
### Job 3: RemoteServer — [SECURITY SPECIALIST]
```

---

### ✅ Upgrade 4: Structured Output Format

Make every agent write findings in the same format so they're easy to merge:

```markdown
### [🔴 CRITICAL / 🟠 IMPORTANT / 🟡 MINOR] Bug Title
- **File:** filename.cs — Line NNN
- **Type:** crash | leak | race | security | logic | polish
- **What happens:** One sentence describing what goes wrong.
- **Why it happens:** One sentence of root cause.
- **How to trigger:** How a user would see this.
- **Already known?** Yes (Bug #NN) / No (NEW)
```

This makes consolidation fast — just copy-paste from all findings files.

---

### ✅ Upgrade 5: Fix → Verify Loop

After bugs are fixed, don't trust the fixer to say "done." Use a **second agent** to verify:

```
Fixer Agent → fixes bug → marks as "fixed"
Verifier Agent → reads the same file → checks if bug is actually gone → confirms or rejects
```

This is called the **Implement-Verify-Fix loop** — industry standard for reliable AI fixing.

The verifier should leave evidence:
- "Checked line 1169 — `ConfigManager.Save()` is now inside `try { }` block. ✅ Confirmed fixed."
- "Checked line 446 — `FlushMacroStats` still has the race condition. ❌ Not fixed."

---

### ✅ Upgrade 6: Dashboard Summary Section

At the bottom of the job board, always keep a live summary table:

```markdown
## 📊 Dashboard

| Job | Area | Status | Agent | Bugs Found | Verified |
|-----|------|--------|-------|------------|---------|
| 1 | Compiler | ✅ DONE | abc123 | 23 | ❌ Not yet |
| 2 | Execution | ✅ DONE | def456 | 15 | ✅ Confirmed |
```

This lets you (the manager) see everything at a glance without reading every file.

---

### ✅ Upgrade 7: Layered Memory (What Each Agent Needs)

Research shows agents work best when they get exactly what they need — no more, no less:

| Layer | File | What It Contains |
|-------|------|-----------------|
| **Global** | `SCHEMA.md` | Project-wide rules, never modify raw/ |
| **Project** | `agent-onboarding.md` | Tech stack, codebase map, do's/don'ts |
| **Task** | `master-scan-jobs.md` | Specific job assignment |
| **Output** | `findings/job{N}.md` | Where to write results |

Agents should read only what they need. Don't paste everything into the prompt.

---

### ✅ Upgrade 8: Prevent Two Agents Grabbing Same Job

The current system uses "first 8 chars of conversation ID" as agent ID. This works! But add a **race condition check**:

After claiming a job, the agent should:
1. Wait 5 seconds
2. Re-read the file
3. Check that THEIR agent ID is still on the job
4. If another agent ID is there → release the job and pick a different one

This prevents two agents accidentally doing the same work.

---

## 🔄 The Full Optimized Workflow

```
ROUND N:
│
├── 1. Manager posts jobs on job board (with type labels)
│
├── 2. Open 10 agents in parallel
│   Each agent reads:
│   - SCHEMA.md (rules)
│   - agent-onboarding.md (project context)
│   - master-scan-jobs.md (grab a job)
│
├── 3. Each agent works independently
│   - Claims job → does work → writes findings → marks DONE
│
├── 4. Verifier agents run (second wave)
│   - Read all findings files
│   - Confirm real bugs vs false positives
│
├── 5. Consolidation (you or a consolidator agent)
│   - Merge all verified findings
│   - Update consolidated-results.md
│   - Update bug-backlog.md
│
└── 6. Review + Approve → Start ROUND N+1
```

---

## 🚫 What NOT To Do (Learned From Research)

- ❌ **Don't make one "god agent"** do everything — it gets overloaded
- ❌ **Don't give agents more than they need** — extra context confuses them
- ❌ **Don't skip verification** — agents can report false bugs
- ❌ **Don't let agents fix things during a scan round** — scan only, fix separately
- ❌ **Don't have agents self-verify** — use a separate agent for verification

---

## 📁 File Structure for This System

```
Obsidian Vault/
  wiki/
    status/
      master-scan-jobs.md        ← Job board (THE central file)
      consolidated-scan-results.md ← Merged findings
      bug-backlog.md             ← Official bug tracker
      findings/
        job1-compiler.md         ← Per-agent output
        job2-execution.md
        ...
  skills/
    multi-agent-system.skill.md  ← This file
  wiki/
    guides/
      agent-onboarding.md        ← What every new agent reads first
```

---

## Related Pages
- [[master-scan-jobs]]
- [[consolidated-scan-results]]
- [[agent-onboarding]]
- [[bug-backlog]]

---

## 🏆 Mode 2: Sprint Pipeline (Bug Bash → Triage → Fix → Verify)

The multi-agent system can also run in **Sprint Pipeline Mode** — a fast, batch-oriented workflow for testing and fixing bugs.

### When to Use Sprint Pipeline vs. Scan Mode

| Use Case | Mode |
|----------|------|
| Deep codebase audit for hidden bugs | Scan Mode (existing) |
| Testing the app and fixing found bugs fast | Sprint Pipeline Mode |
| Pre-release QA sprint | Sprint Pipeline Mode |
| Security or performance review | Scan Mode (existing) |

### Sprint Pipeline Roles

| Role | Who | What They Do |
|------|-----|-------------|
| **Tester** | 👤 Human | Bug Bash sprint — tests app, captures bugs |
| **Manager** | 🧠 Manager Agent | Triages, groups, creates Sprint Spec, tracks progress |
| **Worker** | ⚙️ Worker Agent(s) | Reads Sprint Spec, batch fixes, builds, reports |

### Sprint Pipeline Flow

```
ROUND N:
│
├── 1. Human runs Bug Bash (20 min, quick notes)
│
├── 2. Human dumps ALL bugs to Manager (one message)
│
├── 3. Manager auto-triages:
│   - Classify severity (🔴 > 🟡 > 🟢)
│   - Group by component
│   - Detect shared root causes
│   - Create Sprint Spec in QA_Testing/sprints/
│
├── 4. Manager shows plan → Human approves
│
├── 5. Worker reads Sprint Spec → batch fixes by group
│   - Critical first → Important → Minor
│   - ONE dotnet build at end
│   - Reports back to Manager
│
├── 6. Manager generates targeted verification checklist
│   - Only items that were fixed
│   - Saved in QA_Testing/verify/
│
├── 7. Human verifies → failures go to Round N+1
│
└── 8. Repeat until zero bugs → Ship! 📦
```

### Key Differences from Scan Mode

| Aspect | Scan Mode | Sprint Pipeline |
|--------|-----------|----------------|
| Goal | Find all bugs | Find AND fix bugs |
| Testing | Agent scans code | Human tests the app |
| Fixing | Separate phase | Integrated in pipeline |
| Rounds | Scan → Verify → Consolidate | Bug Bash → Triage → Fix → Verify |
| Output | Findings files | Sprint Specs + Fix reports |
| File location | `wiki/status/findings/` | `QA_Testing/sprints/` |

### Sprint Pipeline File Structure

```
QA_Testing/
├── sprints/
│   ├── sprint-round-1.md    ← Bugs grouped, prioritized
│   ├── sprint-round-2.md    ← Remaining after Round 1
│   └── sprint-round-3.md    ← Final smoke test
├── verify/
│   ├── verify-round-1.md    ← Re-test checklist
│   └── verify-round-2.md
├── dashboard.md              ← Progress tracker
├── MANAGER_AGENT.md          ← Manager instructions
└── WORKER_AGENT.md           ← Worker instructions
```

### Related Skills
- [[sprint-pipeline]] — Master orchestrator skill
- [[bug-triage]] — Auto-triage classification
- [[targeted-verify]] — Verification checklist generation
- [[qa-testing]] — Bug Bash protocol + QA checklist
- [[bug-fixing]] — Batch fix mode
