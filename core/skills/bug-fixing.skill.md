---
name: bug-fixing
description: Complete verified bug-fix workflow for PowerX Keys. Use for any bug fix — from understanding the bug to implementing the fix, building, verifying, and updating the wiki. Includes the structured bug report format and the implement-verify loop.
tags: [skill, bug-fixing, development, workflow, verification]
date: 2026-06-08
status: active
sources:
  - SCHEMA.md
  - wiki/status/bug-backlog.md
  - wiki/status/surface-scan-consolidated.md
---

# 🐛 Skill: Bug Fixing

Bug fixes in PowerX Keys follow a strict sequence. Skipping steps leads to regressions and outdated documentation. This skill is the complete verified-fix workflow.

## Phase 1: Understand (Before Touching Code)

1. **Read the bug report** — what is the exact symptom? What triggers it?
2. **Check if it's already known** → read `wiki/status/bug-backlog.md` and `wiki/status/surface-scan-consolidated.md`
   - If known: check if there's already a fix in progress (`CHECKPOINT.md`)
   - If duplicate: log it as a duplicate and stop
3. **Read the relevant wiki page** for the component → `wiki/index.md` → follow the link
4. **Read the specific source file section** — find the root cause before writing any fix

## Phase 2: Plan (For Non-Trivial Bugs)

For bugs that require 3+ changes, write a mini-plan:
```
Root cause: [one sentence]
Files to change: [list]
Risk: [what could break?]
Verification: [how will you confirm it's fixed?]
```

## Phase 3: Fix (Surgical Changes Only)

- Change ONLY what fixes the bug
- ❌ No refactoring "while I'm here"
- ❌ No fixing other bugs you notice (log them instead)
- ✅ Match the existing code style exactly
- ✅ If adding a null check or try-catch, keep it minimal

## Phase 4: Build & Verify

```powershell
# After EVERY meaningful code change:
dotnet build "c:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\PowerX_Keys_V2.sln"
```

- **Build must pass** before calling the bug fixed
- If AHK-related: check the generated `.ahk` file in `%DOCUMENTS%/PowerX_Keys/Engine/`
- Trace through the logic — confirm the bug path is now impossible

## Phase 5: Document

### Update Bug Backlog
In `wiki/status/bug-backlog.md` or the relevant scan file, mark the bug:
```markdown
### [Bug Title]
- **Status:** ✅ Fixed — [YYYY-MM-DD]
- **Fix:** [one-sentence description of what was changed]
- **File:** [filename.cs — Line NNN]
```

### Log a New Bug (If Found During Fix)
Use this format in `wiki/status/bug-backlog.md`:
```markdown
### [🔴 CRITICAL / 🟠 IMPORTANT / 🟡 MINOR] Bug Title
- **File:** filename.cs — Line NNN
- **Type:** crash | leak | race | security | logic | polish
- **What happens:** One sentence — the symptom
- **Why it happens:** One sentence — the root cause
- **How to trigger:** User action that causes it
- **Already known?** No (NEW)
- **Status:** 🔵 Open
```

### Update Wiki
- Update the relevant wiki page if the behavior changed
- Append to `wiki/log.md`

## Phase 6: Verify (Use a Different Agent If Possible)

The fixer should NOT self-verify for critical bugs. A second pass should:
1. Re-read the original bug report
2. Check the fix makes sense at the call site
3. Confirm the build passes
4. Leave evidence: "Checked [file] line [N] — [condition] now handles [case]. ✅ Confirmed."

## Common PowerX Keys Bug Patterns

| Pattern | Where to look | Fix approach |
|---------|-------------|-------------|
| AHK script not updating | `ScriptCompilerService.cs` | Find the wrong/missing code generation |
| Setting not persisting | `ConfigManager.cs` + `AppConfig.cs` | Check 500ms debounce save trigger |
| UI not updating | ViewModel property | `OnPropertyChanged()` missing? |
| Crash on null | Any ViewModel/Service | Null-check at the entry point |
| Race condition | Async methods | Use `Dispatcher.Invoke` for UI, lock for shared state |
| Hotkey not registering | `ScriptCompilerService.cs` | Check AHK hotkey string generation |
| Engine not reloading after settings change | `ScriptManager.cs` | Call `Stop()` then `Start()` |

## The Implement-Verify-Fix Loop

```
Fix → Build → Logic trace → Update wiki → (Optional: second agent verify)
```

Never call a bug "fixed" without a passing build AND a logic trace confirming the root cause is gone.

## Batch Fix Mode (Sprint Pipeline)

When working from a Sprint Spec (`QA_Testing/sprints/sprint-round-N.md`), switch to **Batch Fix Mode**. This is different from single-bug fixing.

### How Batch Fix Works

1. **Read the Sprint Spec** — it contains all bugs grouped by component with priorities
2. **Fix by component group** — NOT one bug at a time:
   - Fix ALL bugs in Group A (Critical) first
   - Then ALL bugs in Group B (Important)
   - Then ALL bugs in Group C (Minor)
3. **Root cause awareness** — if the Sprint Spec says 3 bugs share a root cause, fix the root cause ONCE instead of patching 3 times
4. **ONE build at the end** — do NOT run `dotnet build` after every single fix:
   ```powershell
   # Run ONCE after ALL fixes are done:
   dotnet build "C:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\PowerX_Keys_V2\PowerX_Keys_V2.csproj"
   ```
5. **Update Sprint Spec** — after fixing, mark each bug in the Sprint Spec:
   ```markdown
   ### BUG-001: Profile rename doesn't persist
   - **Status:** ✅ Fixed — [YYYY-MM-DD]
   - **Fix:** Added CommitTransaction() call in SaveProfile()
   - **File:** Managers\MacroDatabase.cs — Line 245
   ```
6. **Report to Manager** — list all changes made, files modified, build result

### Key Differences: Single vs. Batch

| Aspect | Single Fix | Batch Fix |
|--------|-----------|-----------|
| Bug source | User report or bug-backlog.md | Sprint Spec file |
| Build | After each fix | ONE at the end |
| Priority | Fix whatever's reported | Critical → Important → Minor |
| Grouping | N/A | Fix by component group |
| Root cause | Individual analysis | Cross-bug root cause check |

### Batch Fix Guardrails
- ❌ Do NOT fix bugs not in the Sprint Spec
- ❌ Do NOT refactor "while I'm here"
- ❌ Do NOT touch Smart Menu code unless explicitly listed
- ✅ If you notice a NEW bug while fixing, log it — don't fix it
- ✅ If a fix breaks another fix in the same batch, undo and note the conflict

