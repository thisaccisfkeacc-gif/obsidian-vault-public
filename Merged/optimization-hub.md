---
tags: [index, optimization, audit, roadmap]
date: 2026-07-23
status: completed
---

# 🚀 PowerX Keys — App Optimization & Audit Hub

> **Purpose:** Single folder for all research, audits, findings, and optimization plans. All agents contribute findings here before any changes are made.

---

## 📋 How This Works

1. **Research phase** — Agents scan codebase and add findings here
2. **Review phase** — User reviews all findings
3. **Backup** — Commit all current changes as checkpoint
4. **Implementation** — Start fixing in priority order

---

## 🧪 Testing & QA

| Document | Location | Description |
|----------|----------|-------------|
| [[testing-checklist]] | Root | Master QA checklist — 652 tests across 23 phases |
| [[Block QA Findings]] | `Archive/` | Block-level QA findings |

---

## 🛡️ Error Handling & AHK Leak Audit

| Finding | Status | Details |
|---------|--------|---------|
| Snippets Script — no OnError handler | ❌ Open | Needs OnError added |
| Sandbox Script — no OnError handler | ❌ Open | Needs OnError added |
| FileLauncher — no try/catch on Run() | ❌ Open | Needs try/catch wrapper |
| MouseClick — no try/catch on Click | ❌ Open | Needs try/catch wrapper |
| SendKeys — no try/catch on Send | ❌ Open | Needs try/catch wrapper |
| Scheduled Tasks — no try/catch | ❌ Open | Needs try/catch wrapper |
| ScreenEvent Watchers — no try/catch | ❌ Open | Needs try/catch wrapper |
| SetVariable — expression can throw | ❌ Open | Needs try/catch wrapper |
| CallMacro — missing file not handled | ❌ Open | Needs file check + try/catch |
| WaitForKey — key hook can fail | ❌ Open | Needs try/catch wrapper |
| Notification — TrayTip can fail | ❌ Open | Needs try/catch wrapper |
| Preview Mode — most blocks unguarded | ❌ Open | Needs try/catch on all blocks |
| Error handlers leak raw AHK internals | ❌ Open | Improve callback |
| Executor stderr not read on crash | ❌ Open | Read before respawn |
| Build compilation — raw errors leak | ❌ Open | Catch + show PowerX message |

---

## 📁 File Splitting (Modularization)

### 🔴 Critical (2000+ lines)

| File | Lines | Split Plan |
|------|-------|------------|
| `RemoteServerService.cs` | 5607 | Core + API + Mobile HTML + Controller HTML |
| `ScriptCompilerService.cs` | 5264 | Core + MouseClick + ImagePixel + WindowLogic + Hotkeys + Helpers |
| `SettingsDashboardView.xaml` | 2855 | Shell + 9 tab UserControls |
| `MacroExecutionService.cs` | 2801 | Core + Mouse + ImagePixel + Keyboard + Window + Logic |
| `ScriptLibraryView.xaml` | 2318 | Shell + Categories + MacroList + HotkeyPanel |
| `MacroItem.cs` | 2261 | MacroStepType + MacroStep + MacroStep.Properties + MacroItem + ActionItem |
| `MacroEditorView.xaml` | 2127 | Shell + Toolbar + StepList + PropertyPanel + ContextMenu |

---

## 🔁 Code Duplication

| Pattern | Where | Fix |
|---------|-------|-----|
| Win32 P/Invoke declarations | MacroExecutionService + RemoteServerService | Shared `NativeMethods.cs` |
| Mouse event constants | Both files | Shared constants |
| Window coordinate offset (×4) | MacroExecutionService | `ResolveWindowOffset()` helper |
| Smart Box AHK generation (×2) | SingleStep.cs | Shared method |
| `_MasterLog` function (×2) | ScriptCompilerService | Emit once |
| AHK header boilerplate (×2) | ScriptCompilerService | `GetAhkHeader()` |
| Fire mode blocks (×4, 90% same) | ScriptCompilerService | Template pattern |
| Library CRUD (×4) | MacroDatabase | Generic `CaptureLibrary<T>` |
| ExtraJson serialize/deserialize | MacroDatabase | Source generators |

---

## 🧠 Complex Methods

| Method | Lines | Problem |
|--------|-------|---------|
| `CompileMasterScript()` | 1700+ | One method, entire AHK engine |
| `HandleRequest()` | 1470 | Giant switch, 20+ endpoints |
| `ExecuteStepAsync()` | 1100 | 18-case switch |
| `CompileSingleStepTestScript()` | 1400+ | 5+ nested scope layers |
| `MacroStep` class | 2300 | 100+ properties |

---

## ⚡ WPF Performance

| Area | What To Do |
|------|-----------|
| UI Virtualization | Enable VirtualizingStackPanel on all long lists |
| Deferred Loading | Load Settings, AI panel, Capture Library lazily |
| Freeze Brushes | Mark all SolidColorBrush as Frozen |
| Reduce Bindings | Use OneWay where TwoWay not needed |
| Flatten Visual Tree | Remove unnecessary nesting |
| Async Operations | Move DB/file I/O to background threads |
| Weak Events | Replace += with WeakEventManager |
| Image Optimization | Compress, use DecodePixelWidth |
| Resource Dictionaries | Move shared styles from App.xaml |

---

## 🏗️ Architecture Improvements

| Improvement | Benefit |
|-------------|---------|
| Strategy Pattern for Step Execution | Easy to add new block types |
| Command Pattern for API Routes | Easy to add new endpoints |
| Generic Capture Library | 4x less code |
| Source Generators | Zero manual copy-paste |
| Dependency Injection | Testable, swappable services |
| Event Aggregator | Loose coupling between ViewModels |
| Unit Tests | Catch regressions |
| CI/CD Pipeline | Auto-build + test |

---

## 📊 Priority Order

| Phase | What | Risk | Effort |
|-------|------|------|--------|
| 1 | Extract magic numbers | 🟢 Low | 🟢 Low |
| 2 | Remove Win32 duplication | 🟢 Low | 🟢 Low |
| 3 | Extract coordinate offset helper | 🟢 Low | 🟢 Low |
| 4 | Split ScriptCompilerService | 🟡 Medium | 🟡 Medium |
| 5 | Split MacroExecutionService | 🟡 Medium | 🟡 Medium |
| 6 | Split RemoteServerService | 🟡 Medium | 🟡 Medium |
| 7 | Split MacroItem.cs | 🟡 Medium | 🟡 Medium |
| 8 | Split XAML into UserControls | 🟡 Medium | 🟡 Medium |
| 9 | Strategy pattern for steps | 🔴 Higher | 🔴 High |
| 10 | WPF performance optimizations | 🟡 Medium | 🟡 Medium |
| 11 | Add unit tests | 🟡 Medium | 🔴 High |
| 12 | Source generators | 🔴 Higher | 🔴 High |

---

## 🔍 Existing Research & Audits

| Document | Location | Topic |
|----------|----------|-------|
| [[security-performance-audit]] | `Archive/` | Deep Codebase Audit (Security, Async Crashes, SQLite, Memory Leaks) |
| [[app-modularization]] | `ideas/` | Modularization ideas |
| [[ideas]] | `ideas/` | General feature ideas |
| [[design-references]] | `ideas/` | Design references |
| [[v3-webview2-rewrite]] | `ideas/` | WebView2 rewrite plan |
| [[GOTCHAS]] | `core/` | Known gotchas |
| [[SCHEMA]] | `core/` | Database schema |
| [[DECISIONS]] | `core/` | Architecture decisions |
| [[SOUL]] | `core/` | App soul/persona |
| [[index]] | `core/wiki/index.md` | Wiki index |
| Block Audits | `core/wiki/audits/` | Individual block audits |
| Feature Docs | `core/wiki/features/` | Feature documentation |
| [[Image Search Bug Investigation]] | Root | Image search bugs |
| [[Image Search Cascade and Smart Box Preview]] | Root | Smart box preview |
| [[Multi-AI Collaboration Guide]] | Root | Multi-agent workflow |
| [[Sandbox to Main App Migration Plan]] | Root | Migration plan |
| [[Payment System Architecture]] | Root | Payment system |

---

## 📝 Notes

- All agents (Antigravity, Kiro, etc.) add findings to this folder
- Before implementing any changes, commit current state as checkpoint
- Work through priority list from top to bottom
- Each phase should be a separate commit for easy rollback
