# 🔍 PowerX Keys — Agent 2: Smart Auditor Spec

> **Role**: Agent 2 (Smart Auditor Agent)  
> **Task**: Inspect Raw Macro JSON Responses, Log Schema Errors, Action Syntax Bugs, & Track Regression  
> **Target Output Log**: `C:\Users\Maaz\Desktop\PHASE2_MACRO_TEST_AUDIT.md`

---

## 📌 Core Responsibilities

1. **Deep JSON & Action Audit**:
   * Inspect generated macro outputs for:
     * Root `PowerXMacro` structure compliance.
     * Valid `TriggerModeString` and `TriggerKey`.
     * Proper `FastSteps` array syntax for Action Types 0 through 42.
     * Correct block nesting (Loop `31`, If `20` / Else `21`, Universal EndTag `22`).
     * Clean placeholder usage and presence of 1-sentence user tips.

2. **Detailed Context Logging**:
   * For every identified issue, record:
     * **Batch # & Prompt #**.
     * **Expected Behavior** vs **Actual AI Output**.
     * **Exact Root Cause** (e.g. invalid action string format, missing EndTag, malformed placeholder).
     * **Suggested Fix / Rule Update**.

3. **High-Level Scorecard Summary**:
   * Maintain a Pass/Fail scorecard index at the top of `PHASE2_MACRO_TEST_AUDIT.md` for instant visual tracking.

4. **Multi-Run Regression Tracking (Tests 1 to 10)**:
   * On subsequent test runs (Test 2, 3, etc.), re-verify that previously fixed issues remain resolved while logging any newly discovered edge cases.
