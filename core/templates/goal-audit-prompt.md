# 🔍 Goal Audit Prompt Template

> **Reusable `/goal` prompt for auditing any feature in the PowerX Keys app.**
> Copy the prompt below, replace `[FEATURE NAME]` and `[FEATURE DESCRIPTION]` with your target, then use it with `/goal` in a new chat session.

---

## How to Use

1. Open a **new chat session**
2. Type `/goal`
3. Paste the prompt below (with your replacements filled in)
4. Send it and walk away — the agent will handle everything non-stop

---

## The Prompt

```
Your job is to fully audit and perfect [FEATURE NAME] in the PowerX Keys WPF app at:
C:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\PowerX_Keys_V2

[FEATURE DESCRIPTION]

Follow these phases strictly in order. Do not skip any phase. Do not stop until all phases are complete.

---

📋 PHASE 1 — PREP (do this once)
- Read Obsidian Vault/SCHEMA.md and wiki/index.md to understand the full codebase structure
- Locate where [FEATURE NAME] is defined, stored, and used in the codebase
- Create a full file checklist: list every .xaml, .cs, and .xaml.cs file in the project into a document called feature_audit.md — every file must be on this list before scanning begins

---

🚫 GIT SAFETY RULES — STRICTLY ENFORCED FOR THIS ENTIRE SESSION
- NEVER run: git revert, git restore, git reset, git checkout, or any command that discards or reverts changes
- The ONLY allowed git-style rollback is manually undoing your own code edits file by file
- If a fix breaks the build after 2 attempts, revert ONLY that specific file edit manually — do NOT use git commands
- Reason: uncommitted work can be permanently lost if git restore/revert is used. This is irreversible.
- If you feel you need to use git, STOP and ask the user instead

---

🔁 PHASE 2 — SCAN LOOP (repeat until checklist is 100% complete)
- Go through the file checklist one file at a time
- For each file, scan it for anything related to [FEATURE NAME]
- Log every finding into feature_audit.md using this exact format:
  | File | Line | Issue | Severity (Low/Medium/High) | Status (Genuine / False Flag) |
- After scanning a file, tick it off the checklist
- Do NOT move to Phase 3 until every single file on the checklist is ticked off
- If you discover new files during scanning that were not on the original list, add them and scan those too

---

🔍 PHASE 3 — FILTER (3 mandatory passes, do not skip any)

Pass 1 — First sweep: Go through every finding. Mark anything obvious as [FALSE FLAG] with a reason. Everything else stays.

Pass 2 — Stress test: For each remaining finding, ask yourself: "If I don't fix this, will it cause a real bug or real problem for the user?" If the honest answer is no — mark it [FALSE FLAG]. Only keep things that are genuinely risky or broken.

Pass 3 — Final confidence check: Read the remaining list one more time from top to bottom. For each item, rate your confidence that it needs fixing: Low / Medium / High. Anything with Low confidence gets marked [FALSE FLAG]. Only items with Medium or High confidence get marked [NEEDS FIX].

After all 3 passes, the [NEEDS FIX] list is final. Write a short reason next to each decision so it is clear why.

---

🔧 PHASE 4 — FIX
- Fix every [NEEDS FIX] item one by one, in order of severity (High first)
- STRICT SCOPE RULE: Only fix things directly related to [FEATURE NAME]. Do NOT refactor, rename, or touch anything outside the scope of this audit
- OUT-OF-SCOPE BUGS: If you accidentally discover a bug that is NOT related to [FEATURE NAME], do NOT fix it. Instead, log it in a separate section at the bottom of feature_audit.md called "🐛 Other Bugs Found" with the file name, line number, and a short description. These can be reviewed and fixed in a separate session later.
- After every single file change, run: dotnet build "PowerX_Keys_V2.csproj"
- If the build passes, move to the next fix
- If the build fails, attempt to fix the error. If it still fails after 2 attempts, revert that specific change, mark it as [SKIPPED — BUILD FAILED] in feature_audit.md, and continue to the next item

---

📝 PHASE 5 — WRAP UP
- Update feature_audit.md with a final summary section listing:
  - Total files scanned
  - Total findings
  - Total fixed
  - Total skipped
  - Total false flags
- Update the relevant wiki pages in wiki/ to reflect any changes made
- Log all changes in wiki/log.md with today's date
- **IMPORTANT:** Open `Obsidian Vault/audit-areas.md` and update the Status column for this area from ⬜ to ✅ Done (or 🔧 Scanned if fixes were found but not all applied). This creates a permanent record that this area has been covered.
```

---

## Example Replacements

| Placeholder | Example Value |
|---|---|
| `[FEATURE NAME]` | Performance Mode |
| `[FEATURE DESCRIPTION]` | This feature helps low-end PC users by disabling heavy visual effects, animations, and resource-intensive UI elements when enabled. |

---

## Notes

- The document `feature_audit.md` will be created by the agent in the project root during the audit
- This prompt is designed to be **safe** — it has a strict scope rule and a rollback plan for broken builds
- Works for any feature, not just Performance Mode — just swap the placeholders
