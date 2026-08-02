---
name: context-compression
description: Handle long agent sessions by summarizing context, updating MEMORY.md, and resetting working memory without losing critical state. Use when your session has been running a long time, you've read many files, or you need to hand off work to a new agent.
tags: [skill, context, compression, memory, session]
date: 2026-06-08
status: active
---

# 🗜️ Skill: Context Compression

Long sessions degrade agent performance. When context grows large, reasoning quality drops and the agent starts "forgetting" earlier instructions. This skill teaches you to compress intelligently — not truncate blindly.

## When to Use This Skill

Trigger context compression when:
- You've been working for a long session with many tool calls
- You've read 5+ source files
- You feel like you might be losing track of the original task
- You're about to hand off to a new agent via CHECKPOINT.md
- You want to "save progress" before tackling the next phase

## The Write-Select-Compress-Isolate Loop

```
1. WRITE: Externalize state to files (CHECKPOINT.md, MEMORY.md)
2. SELECT: Identify the 3-5 most important facts for the next step
3. COMPRESS: Summarize completed work in 1-2 sentences each
4. ISOLATE: Separate what you know (semantic) from what you did (episodic)
```

## Step-by-Step Compression Protocol

### Step 1: Externalize Everything Important

Before compressing, write to permanent files:

**Update CHECKPOINT.md** (if mid-task):
```markdown
## [ACTIVE CHECKPOINT]
- Task: [what you're doing]
- Last completed step: [description]
- Next step: [exact instruction for next agent]
- Files touched: [list]
- Context needed: [any critical state]
```

**Append to MEMORY.md Layer 2** (episodic log):
```markdown
## [YYYY-MM-DD] — [Task title]
- Did: [summary]
- Key files: [list]
- Decision: [any choices made]
- Status: 🔵 In Progress
- Next: [what to do next]
```

**Update MEMORY.md Layer 3** if you discovered new project facts.

### Step 2: Identify What the Next Step Truly Needs

Ask yourself: "If a fresh agent reads CHECKPOINT.md, MEMORY.md, and the relevant wiki page — will they have enough to continue?"

If yes → compression is safe.  
If no → add the missing facts to one of those files first.

### Step 3: Prune Tool Outputs from Mental Context

After externalizing, mentally treat these as "done and filed":
- Old file reads you're no longer referencing
- Tool outputs from completed steps
- Any intermediate search results

Keep only:
- The current task goal
- The immediate next 1-2 steps
- Any blocking constraints

### Step 4: Continue or Hand Off

**If continuing:** You now have a compressed working context. Proceed.

**If handing off:** Ensure CHECKPOINT.md is fully updated, then tell the user to open a new agent session. The new agent will:
1. Read AGENTS.md (auto-loaded)
2. Read SOUL.md + MEMORY.md
3. Read CHECKPOINT.md → pick up exactly where you left off

## What NOT to Compress Away

Never lose:
- The root cause of a bug you found (write it to wiki if needed)
- A decision you made and why (write to DECISIONS.md)
- Build errors you haven't resolved yet
- The exact next step needed (write to CHECKPOINT.md)

## Compression Signals from Research

| Signal | Meaning | Response |
|--------|---------|----------|
| Repeating yourself | Context is bloated | Compress and continue |
| Forgetting earlier constraints | Context drift | Re-read SOUL.md + AGENTS.md |
| Output getting vaguer | Context near limit | Save state → fresh session |
| Confusing two different bugs | Mixed context | Write each to wiki → reset |

## MEMORY.md Compression Rule

Keep MEMORY.md Layer 2 entries brief. If the file is getting long (20+ entries), summarize the oldest entries:

```markdown
## [ARCHIVED] Summary through YYYY-MM-DD
- [3-5 bullet summary of everything before this date]
- [Major decisions made]
- [Major bugs fixed]
- [Current project state at this point]
```

Then mark the summarized entries as `[archived]` — don't delete them.
