---
name: bug-triage
description: Classify and group bugs by severity, component, and root cause. Use when processing raw bug reports from a Bug Bash sprint into structured, prioritized groups ready for batch fixing.
tags: [skill, triage, bugs, classification, severity, grouping]
date: 2026-06-30
status: active
sources:
  - wiki/status/bug-backlog.md
  - wiki/status/known-issues.md
  - wiki/status/testing-checklist.md
---

# 🏷️ Skill: Bug Triage (Auto-Classification)

**Summary:** Takes raw bug notes from a Bug Bash and transforms them into structured, severity-classified, component-grouped bug reports with root-cause detection. This is the "Think" phase of the OTAV cycle.

---

## 🎯 Severity Classification

Every bug gets ONE severity level:

| Severity | Icon | Criteria | Examples |
|----------|------|----------|----------|
| **Critical** | 🔴 | Crashes, data loss, blocks usage entirely | App crashes on launch, profiles deleted on save, hotkeys stop working |
| **Important** | 🟡 | Wrong behavior, missing functionality, broken workflows | Settings don't persist, macro playback skips steps, search returns wrong results |
| **Minor** | 🟢 | Cosmetic issues, UI polish, typos, visual glitches | Button misaligned, wrong tooltip text, icon color off |

### Priority Order:
```
🔴 Critical  →  fix FIRST (blocks the user)
🟡 Important →  fix SECOND (things work but wrong)
🟢 Minor     →  fix LAST (polish and cleanup)
```

---

## 🗂️ Component Grouping

Group bugs that affect the **same files or functional area** together. Fixing them as a batch is faster and reduces build cycles.

### PowerX Keys Components:

| Component | Key Files | Area |
|-----------|----------|------|
| **Profile System** | `ProfileViewModel.cs`, `ProfileManager.cs`, `Profile.cs` | Profile CRUD, switching, import/export |
| **Macro Editor** | `MacroEditorViewModel.cs`, `MacroEditorView.xaml` | Editing macros, cursor, syntax |
| **Settings** | `SettingsViewModel.cs`, `ConfigManager.cs`, `AppConfig.cs` | All app settings and persistence |
| **Script Library** | `ScriptLibraryViewModel.cs`, `ScriptLibraryView.xaml` | Script list, search, filtering |
| **Script Compiler** | `ScriptCompilerService.cs` | AHK script generation |
| **Engine** | `ScriptManager.cs`, `ProcessManager.cs` | Engine start/stop/reload |
| **Hotkey System** | `HotkeyEditorViewModel.cs`, `HotkeyService.cs` | Hotkey binding, registration, conflicts |
| **UI Shell** | `MainWindow.xaml`, `NavigationService.cs` | Navigation, layout, window management |
| **Smart Menu** | `SmartMenu*.cs`, `SmartMenuView.xaml` | Clipboard manager (⚠️ do not modify unless asked) |

### Grouping Rules:
- If 2+ bugs affect the **same file** → group them
- If 2+ bugs affect the **same ViewModel** → group them
- If bugs span multiple components but share a **single root cause** → flag as root-cause group

---

## 🔍 Root Cause Detection

When multiple bugs might share an underlying cause, flag them:

### Signs of a Shared Root Cause:
- Multiple bugs reference the **same file and method**
- Multiple bugs trigger under the **same user action** (e.g., "after switching profiles")
- Multiple bugs involve the **same data flow** (e.g., settings save pipeline)
- A single missing null-check or event handler could cause several symptoms

### Root Cause Group Format:
```markdown
## ⚠️ Potential Root-Cause Groups

### Group A: ConfigManager Save Pipeline
- BUG-003: Settings don't persist after restart
- BUG-007: Theme resets on app launch
- BUG-011: Custom hotkey labels disappear
→ Likely root cause: ConfigManager.Save() not flushing all properties
→ Fix one, re-test all three

### Group B: Profile Switch Event
- BUG-004: Hotkeys don't reload after profile switch
- BUG-009: Macro list shows wrong profile's macros
→ Likely root cause: ProfileChanged event not firing or not handled
→ Fix one, re-test both
```

---

## 📝 Structured Bug Template

Every triaged bug MUST use this format:

```markdown
🐛 BUG-NNN | 🔴/🟡/🟢 Severity | Component Name
→ Description of the bug
→ Steps to reproduce (if known)
→ Component: relevant ViewModel/Service/View
→ Affected files: file paths
```

### Example:

```markdown
🐛 BUG-001 | 🔴 Critical | Profile System
→ Switching profiles does not reload hotkeys — old profile's hotkeys remain active
→ Steps: Create 2 profiles with different hotkeys → switch from Profile A to Profile B → press Profile B's hotkey
→ Component: ProfileViewModel.cs, ScriptManager.cs
→ Affected files: ViewModels/ProfileViewModel.cs, Services/ScriptManager.cs

🐛 BUG-002 | 🟡 Important | Settings
→ Theme dropdown selection doesn't persist after app restart
→ Steps: Open Settings → change theme to Dark → close and reopen app → theme is back to Light
→ Component: SettingsViewModel.cs, ConfigManager.cs
→ Affected files: ViewModels/SettingsViewModel.cs, Services/ConfigManager.cs

🐛 BUG-003 | 🟢 Minor | Macro Editor
→ Line numbers in macro editor are slightly misaligned with code text
→ Steps: Open any macro in the editor → look at line numbers vs. code lines
→ Component: MacroEditorView.xaml
→ Affected files: Views/MacroEditorView.xaml
```

---

## 📋 Triage Output Format

After triaging, produce a structured report:

```markdown
# 🏷️ Triage Report — [Date]

**Source:** Bug Bash Round N
**Total bugs found:** X
**Breakdown:** 🔴 A | 🟡 B | 🟢 C

---

## 🔴 Critical (fix first)

### Component: [Name]
🐛 BUG-001 | 🔴 Critical | ...
🐛 BUG-002 | 🔴 Critical | ...

---

## 🟡 Important (fix second)

### Component: [Name]
🐛 BUG-003 | 🟡 Important | ...

---

## 🟢 Minor (fix last)

### Component: [Name]
🐛 BUG-004 | 🟢 Minor | ...

---

## ⚠️ Potential Root-Cause Groups
[see root cause format above]

---

## 🚫 Duplicates Skipped
- "Theme not saving" → already in known-issues.md as ISSUE-042
- "App slow on launch" → already in bug-backlog.md as BUG-108
```

---

## 🔢 Bug Numbering

- Use sequential numbering within each sprint round: `BUG-001`, `BUG-002`, etc.
- Sprint round prefix is optional: `R1-BUG-001` for Round 1
- Do NOT reuse numbers from previous rounds

---

## 🚫 Duplicate Detection

Before adding a bug to the triage report, check:

1. `wiki/status/known-issues.md` — is this already a known issue?
2. `wiki/status/bug-backlog.md` — was this already reported?
3. Previous sprint specs in `QA_Testing/sprints/` — was this in a prior round?

If duplicate: skip it and note in the "Duplicates Skipped" section.

---

## Related Pages

- [[sprint-pipeline]] — The master pipeline that uses this skill
- [[targeted-verify]] — Verification after fixes are applied
- [[bug-fixing]] — The verified bug-fix workflow
- [[qa-testing]] — The 70-test QA checklist
- [[bug-backlog]] — Official bug tracker
- [[known-issues]] — Known issues reference
