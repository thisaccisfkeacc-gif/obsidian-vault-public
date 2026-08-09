---
name: qa-testing
description: Run and update the PowerX Keys 70-test QA checklist. Use when verifying a fix, preparing for a release, running regression tests, or adding new test cases to the checklist.
tags: [skill, qa, testing, quality, checklist]
date: 2026-06-08
status: active
sources:
  - wiki/status/testing-checklist.md
---

# 🧪 Skill: QA Testing

PowerX Keys has a 70-test QA checklist. This skill covers how to run tests, interpret results, add new tests, and update the checklist.

## The QA Checklist Location

**File:** `wiki/status/testing-checklist.md`

**Current status:** 13 passed, 53 untested (as of 2026-06-08)

## Running Tests

Tests are manual — PowerX Keys doesn't have automated unit tests for UI flows. Each test is a user-actionable step with a verifiable outcome.

### How to Run a Test

1. Open `wiki/status/testing-checklist.md`
2. Find the test you want to run
3. Perform the actions described
4. Record the result using the status markers:

```markdown
| Test | Status | Notes |
|------|--------|-------|
| TC-001: Launch app | ✅ Pass | Opens to Script Library |
| TC-002: Record macro | ❌ Fail | Recording stops after 3s (bug) |
| TC-003: Save settings | ⏭️ Skip | Blocked by TC-002 failure |
| TC-004: Export macro | 🔵 Untested | |
```

### Status Markers

| Marker | Meaning |
|--------|---------|
| ✅ Pass | Test passed, behavior is correct |
| ❌ Fail | Test failed — log a bug |
| ⚠️ Partial | Partially works — note what broke |
| ⏭️ Skip | Blocked by another failure or not applicable |
| 🔵 Untested | Not yet run |

## Bug Discovery During Testing

When a test fails:

1. Note the failure in the checklist with details
2. Create a bug entry in `wiki/status/bug-backlog.md`:
   ```markdown
   ### [🔴/🟠/🟡] Bug Title
   - **Triggered by:** Test TC-NNN
   - **File:** (if known)
   - **What happens:** symptom
   - **How to reproduce:** exact steps
   - **Status:** 🔵 Open
   ```
3. Do NOT fix the bug during a QA pass — scan first, fix separately

## Adding New Test Cases

When a new feature is shipped, add test cases. Format:

```markdown
### TC-NNN: [Feature/Area] — [What is being tested]
- **Prerequisites:** What must be set up first
- **Steps:**
  1. Do this
  2. Then this
  3. Check this
- **Expected:** What should happen
- **Status:** 🔵 Untested
- **Added:** YYYY-MM-DD (feature: [name])
```

**Numbering:** Use the next sequential TC number.

## Test Categories

| Category | Test Range (approx) | Focus |
|----------|--------------------|----|
| Core Launch | TC-001 to TC-010 | App starts, closes, navigates |
| Macro Recording | TC-011 to TC-020 | Record, playback, save |
| Hotkeys | TC-021 to TC-030 | Bind, trigger, modify |
| SmartMenu | TC-031 to TC-045 | All SmartMenu features |
| Settings | TC-046 to TC-055 | All settings persist correctly |
| Import/Export | TC-056 to TC-063 | .pxmacro format |
| Edge Cases | TC-064 to TC-070 | Empty states, large data, errors |

## QA Pass Strategy

For a full QA pass before release:

1. Run all ✅ Pass tests first — confirm regressions haven't appeared
2. Run untested cases in category order
3. Stop and log bugs when found (don't debug, just log)
4. After scan: prioritize failures by severity
5. Fix 🔴 CRITICAL failures first
6. Re-run affected tests after fixes

## Update the Checklist After Each Session

At end of any QA work:
- Update test statuses in `wiki/status/testing-checklist.md`
- Append to `wiki/log.md`
- Append to `MEMORY.md` Layer 2 with QA session summary

## Bug Bash Sprint Protocol (Fast Batch Testing)

For rapid full-app testing, use the **Bug Bash Sprint** instead of testing one feature at a time.

### What is a Bug Bash?
A time-boxed session (20-30 min) where you test ALL features fast and collect ALL bugs at once — without stopping to fix anything.

### How to Run a Bug Bash Sprint

1. **Set a timer** — 20-30 minutes max
2. **Open the app** and go through the checklist category by category
3. **For each bug found**, write a **quick one-liner** only:
   - ❌ Don't write detailed reports during the bash
   - ❌ Don't stop to investigate root causes
   - ❌ Don't try to fix anything
   - ✅ Just note what broke and move on
4. **After the timer ends**, dump ALL bugs to the Manager agent in one message
5. The Manager will structure, classify, and group them

### Quick Note Format (During Sprint)
```
- "profile rename doesn't save after restart"
- "editor crashes when dragging empty step"
- "hotkey F5 doesn't trigger macro"
- "dark mode toggle has wrong color"
```

### After the Bug Bash
The Manager agent will:
1. Structure each bug with severity and component
2. Group related bugs together
3. Create a Sprint Spec in `QA_Testing/sprints/sprint-round-N.md`
4. Show you the plan for approval
5. Hand off to Worker for batch fixing

### Key Rules
- ⏱️ **Time-boxed** — stop when timer ends, even if untested items remain
- 🚫 **No fixing** — capture only, fix later in batch
- 📋 **Use the checklist** — `wiki/status/testing-checklist.md` as your guide
- 📝 **Quick notes only** — one line per bug is enough

## Related Pages

- [[reports/testing-checklist]] — The actual 70-test checklist
- [[bug-backlog]] — Where discovered bugs are tracked
- [[pre-ship-critical-bugs]] — Bugs that must be fixed before packaging
- [[sprint-pipeline]] — The full Sprint Pipeline orchestration skill
