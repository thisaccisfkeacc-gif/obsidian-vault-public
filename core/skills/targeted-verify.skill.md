---
name: targeted-verify
description: Generate targeted re-test checklists from sprint specs. Use after a batch fix round to verify ONLY the items that were fixed, track results, and feed failures into the next sprint round.
tags: [skill, verification, checklist, re-test, targeted, sprint]
date: 2026-06-30
status: active
sources:
  - QA_Testing/sprints/
  - QA_Testing/verify/
  - QA_Testing/dashboard.md
  - wiki/status/testing-checklist.md
---

# ✅ Skill: Targeted Verification

**Summary:** After a batch fix round, generate a verification checklist containing ONLY the fixed items. The user re-tests just what was fixed — no full regression pass needed. Results feed into the next sprint round or close out the pipeline.

---

## 🎯 Core Principle

**Don't re-test everything. Re-test ONLY what was fixed.**

A full QA pass has 70+ tests. After fixing 8 bugs, the user only needs to verify those 8 fixes. This keeps rounds fast and focused.

---

## 📝 Generating a Verification Checklist

### Input:
- The sprint spec from `QA_Testing/sprints/sprint-round-N.md`
- The Worker's fix log (what was actually changed)

### Process:
1. Read the sprint spec — get the list of all bugs that were in scope
2. Read the Worker's changes — confirm which bugs were actually addressed
3. For each fixed bug, write **exact verification steps** the user can follow
4. Skip any bugs the Worker deferred or couldn't fix (they'll go to Round N+1 automatically)
5. Save the checklist to `QA_Testing/verify/verify-round-N.md`

---

## 📋 Verification Checklist Format

```markdown
# ✅ VERIFY ROUND N — Re-test These Items

**Sprint spec:** QA_Testing/sprints/sprint-round-N.md
**Date:** YYYY-MM-DD
**Total items to verify:** X

---

## 🔴 Critical Fixes

- [ ] BUG-001: Switch profiles → hotkeys should now reload immediately
  → Expected: Profile B's hotkeys work after switching from Profile A
  → Status: ___

- [ ] BUG-002: Open app after crash → no data loss in saved macros
  → Expected: All macros from last session are present
  → Status: ___

---

## 🟡 Important Fixes

- [ ] BUG-003: Change theme in Settings → restart app → theme persists
  → Expected: Dark theme is still active after restart
  → Status: ___

- [ ] BUG-004: Record macro → save → cursor stays on current line
  → Expected: Cursor does not jump to line 1 after saving
  → Status: ___

---

## 🟢 Minor Fixes

- [ ] BUG-005: Open macro editor → line numbers align with code
  → Expected: Line 10's number aligns with line 10's code text
  → Status: ___

---

## ⏭️ Deferred (Not Fixed This Round)

- BUG-006: [reason for deferral] → moves to Round N+1

---

## 📊 Round Results

| Metric | Count |
|--------|-------|
| Total verified | ___ |
| ✅ Fixed | ___ |
| ❌ Still broken | ___ |
| 🆕 New bugs found | ___ |
| ⏭️ Deferred | ___ |
```

---

## 🏷️ Status Markers

After testing each item, the user marks it with one of these:

| Marker | Meaning | What Happens Next |
|--------|---------|-------------------|
| ✅ **Fixed** | Bug is confirmed gone | Mark as resolved in checklist and dashboard |
| ❌ **Still broken** | Bug persists despite fix attempt | Goes to Round N+1 sprint spec |
| 🆕 **New bug found** | A new issue appeared (possibly regression) | Triage via [[bug-triage]], add to Round N+1 |

---

## 🆕 Handling New Bugs During Verification

When the user discovers a NEW bug while verifying fixes:

1. User notes it briefly: `🆕 New: [description]`
2. Manager triages the new bug using [[bug-triage]] (severity, component, root cause check)
3. New bug gets a fresh bug number in the next round
4. It goes into `QA_Testing/sprints/sprint-round-(N+1).md`
5. If the new bug is 🔴 Critical, flag it immediately — don't wait for the next round

### New Bug Quick-Log Format:
```markdown
🆕 NEW during Round N verification:
→ Description: [what happened]
→ While testing: BUG-NNN (the fix I was verifying)
→ Severity estimate: 🔴/🟡/🟢
```

---

## 📊 Updating the Dashboard

After each verification round, update `QA_Testing/dashboard.md`:

```markdown
# 📊 Sprint Pipeline Dashboard

## Round History

| Round | Date | Bugs In | ✅ Fixed | ❌ Broken | 🆕 New | Remaining |
|-------|------|---------|----------|-----------|--------|-----------|
| 1 | 2026-06-30 | 12 | 9 | 2 | 1 | 3 |
| 2 | 2026-07-01 | 3 | 3 | 0 | 0 | 0 |

## Current Status: 🎉 All Clear (or 🔄 Round N+1 needed)

## Bugs Still Open
- ❌ BUG-004: [brief description] — still broken from Round 1
- 🆕 BUG-013: [brief description] — new from Round 1 verification
```

---

## 📋 Updating the Master Testing Checklist

After verification, sync results back to `wiki/status/testing-checklist.md`:

1. Find the test cases that correspond to each verified bug
2. Update their status:
   - ✅ Fixed bug → test case status becomes `✅ Pass`
   - ❌ Still broken → test case stays `❌ Fail`
   - 🆕 New bug → add a NEW test case if one doesn't exist
3. Add notes with the sprint round reference: `Fixed in Sprint Round N`

### Example Update:
```markdown
| TC-023 | Profile hotkey reload | ✅ Pass | Fixed in Sprint Round 1 (BUG-001) |
| TC-024 | Theme persistence | ✅ Pass | Fixed in Sprint Round 1 (BUG-003) |
| TC-025 | Macro save cursor | ❌ Fail | Still broken — Sprint Round 2 |
```

---

## 🔁 Feeding Into the Next Round

After verification is complete:

1. Collect all ❌ **Still broken** items
2. Collect all 🆕 **New bugs** (already triaged)
3. Combine into `QA_Testing/sprints/sprint-round-(N+1).md`
4. The next round should be **smaller** than the current one
5. If Round N+1 has **zero items** → the pipeline is complete 🎉

### Shrinking Check:
```
Round 1: 12 bugs
Round 2: 3 bugs   ← good, shrinking ✅
Round 3: 0 bugs   ← done! 🎉

⚠️ If a round has MORE bugs than the previous round,
   something is wrong — stop and investigate before continuing.
```

---

## 📁 File Locations

| File | Path | Purpose |
|------|------|---------|
| Verification checklists | `QA_Testing/verify/verify-round-N.md` | Per-round re-test checklist |
| Sprint specs | `QA_Testing/sprints/sprint-round-N.md` | Input — what was fixed |
| Dashboard | `QA_Testing/dashboard.md` | Overall progress tracker |
| Master checklist | `wiki/status/testing-checklist.md` | Sync verified results here |

---

## Related Pages

- [[sprint-pipeline]] — The master pipeline that orchestrates this skill
- [[bug-triage]] — Classification and grouping of bugs
- [[bug-fixing]] — The verified bug-fix workflow
- [[qa-testing]] — The 70-test QA checklist and testing workflow
- [[reports/testing-checklist]] — Master QA checklist
