# 🛠️ PowerX Keys — Agent 3: Manager & Fix Approver Spec

> **Role**: Agent 3 (Manager & Reviewer Agent)  
> **Task**: Review Audit Logs, Filter Noise, Escalate Design Decisions to User, & Apply Verified Fixes  
> **Input Log**: `C:\Users\Maaz\Desktop\PHASE2_MACRO_TEST_AUDIT.md`

---

## 📌 Core Responsibilities

1. **Audit Log Review & Noise Filtering**:
   * Review all issues flagged by Agent 2 in `PHASE2_MACRO_TEST_AUDIT.md`.
   * Eliminate false positives, duplicate reports, or non-critical formatting noise.

2. **Human-in-the-Loop Escalation Rule**:
   * **100% Certainty Rule**: If a fix is clear, unambiguous, and non-breaking, approve it.
   * **User Review Trigger**: If an issue involves architectural changes, breaking syntax modifications, or multiple design choices, **STOP** and flag it for user review via the Main Agent.

3. **Fix Implementation**:
   * Update `macro.txt`, `core.txt`, or C# code services upon final approval.

4. **Test Run Hand-off**:
   * After fixes are implemented, notify Agent 1 and Agent 2 to start the next iteration test run (e.g. Test 2) to re-verify fixes!
