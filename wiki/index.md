---
tags: [wiki, index]
date: 2026-07-15
status: active
---

# 📋 PowerX Keys — Wiki Index

> **Start here.** This is the master catalog of all wiki pages. Read [[agent-onboarding]] first if you're a new AI agent.

---

## 🏗️ Architecture

| Page | Description |
|------|-------------|
| [[overview]] | Tech stack, project structure, MVVM pattern, key architectural insights |
| [[execution-pipeline]] | JSON config → ScriptCompilerService → AHK script → PowerX_Engine.exe flow |
| [[dual-execution-model]] | When macros use AHK vs C# P/Invoke execution and why both exist |
| [[component-relationships]] | How MainWindow → Views → ViewModels → Services → AHK Engine connect |

---

## ⚙️ Services

| Page | Description |
|------|-------------|
| [[script-compiler]] | The brain — generates AHK scripts from JSON config (163KB, largest service) |
| [[macro-execution]] | C#-side macro execution via Win32 P/Invoke (mouse, keyboard, windows) |
| [[remote-server]] | Embedded HTTP server for mobile remote control with PIN auth & QR code |
| [[macro-recording]] | Records mouse/keyboard input via AHK subprocess, outputs JSON |
| [[config-manager]] | JSON settings persistence with 500ms debounced auto-save |
| [[smooth-trace-engine]] | Mouse path playback with Catmull-Rom spline smoothing (3 physics profiles) |
| [[find-text]] | Pixel/image pattern matching using Feiyue's FindText library |
| [[ui-element-capture]] | Windows UI Automation element capture with CTRL-lock |
| [[auto-update]] | Checks GitHub Pages for updates, launches PowerX_Updater |
| [[ai-assistant]] | Built-in free AI with multi-provider fallback chain (no login required) |
| [[master_free_api_keys_catalog]] | 🔑 Master catalog of free AI API keys, rates, daily quotas & rotation strategies |
| [[shortcut-manager]] | Manager for customizable editor keyboard shortcuts (undo, redo, move steps, etc.) |
| [[telemetry]] | Silent anonymous telemetry service for tracking DAU & website downloads |
| [[file-search]] | The file search service — manages Everything background process checks and safe SDK queries |

---

## 🎯 Features

| Page | Description |
|------|-------------|
| [[macro-editor]] | The macro builder/timeline UI — drag-drop, recording, undo/redo |
| [[script-library]] | Main dashboard — macro library, folders, hotkey bindings, visual keyboard |
| [[settings-dashboard]] | All 26+ settings, factory reset, performance mode |
| [[onboarding-guide]] | 🧭 First-time user guide cards — Quick Actions, My Macros, Text Snippets (3-step each) |
| [[image-recognition]] | FindText visual matching — Gray/Color/GrayDiff modes |
| [[mouse-trace]] | Trace recording & playback with physics-based smoothing |
| [[mobile-remote]] | Mobile remote control — 15+ API endpoints, volume, soundboard |
| [[ai-chat]] | In-app AI assistant — Chat/Build modes, macro injection |
| [[easter-eggs]] | 🥚 Hidden secrets & achievements — confetti, sound, animations, multiple trigger types |
| [[schedule-trigger]] | Upgraded time-of-day and day-of-week macro scheduling triggers |
| [[app-blocking]] | 🚫 Generic App Blocking (PowerX Block / Reef) — complete blocks or windowed time limits |
| [[app-time-limiter]] | Windowed time limits for blocked apps |
| [[capture-library]] | 📂 Capture Library (Elements, Images, Pixels) — preview, select, and reuse captured history |
| [[dynamic-visibility-audit]] | 🔍 Audit of dynamically appearing/hiding UI options — gray-out migration candidates |
| [[smart-menu-ui-ux]] | Unified visual and interaction design guide for the Smart Menu |
| [[smart-desktop-click]] | Smart desktop click feature |
| [[smart-image-capture-limit]] | Smart image capture limit logic |
| [[smart-mode-audit]] | Smart Mode edge cases — emoji/multi-byte, modifier wrap, empty backspace |
| [[undo-redo-audit]] | 🔍 Edge case audit of UndoRedoService — 2 critical, 4 medium findings |
| [[error-handling-cleanup]] | AHK error dialog suppression + themed popup replacement plan |
| [[pinpoint-capture-investigation]] | Pinpoint capture feature investigation |
| [[subscription-migration-removal-list]] | Subscription/migration code removal tracking |

---

## 📦 Models

| Page | Description |
|------|-------------|
| [[app-config]] | SettingsModel (26+ properties), ActionItem, TriggerMode enum |
| [[macro-item]] | MacroItem, MacroStep, MacroStepType enum, all step properties |
| [[database-schema]] | SQLite schema, JSON blob design, CRUD operations, migration |

---

## 🔧 Managers

| Page | Description |
|------|-------------|
| [[script-manager]] | AHK process lifecycle — 4 micro-services (master, macro, tester, recorder) |
| [[macro-transfer-manager]] | Import/export system — .pxmacro format, media bundling |
| [[ai-login-manager]] | OpenRouter PKCE OAuth2 login flow (dead code — reserved for future Gmail gate) |

---

## 📊 Status

| Page | Description |
|------|-------------|
| [[known-issues]] | 75+ bugs fixed ✅ — low-severity bugs deferred to post-launch update |
| [[future-plans]] | Parked ideas — pre-shipped cards, window resize presets |

---

## 📚 Guides

| Page | Description |
|------|-------------|
| [[agent-onboarding]] | ⭐ **READ THIS FIRST** — Project overview, wiki usage, do's and don'ts |
| [[adding-a-feature]] | Step-by-step for adding new step types, settings, and features |
| [[packaging-and-obfuscation]] | 📦 Production release compilation, code scrambling, and packaging workflow |
| [[win32-keyboard-gotchas]] | ⚠️ **Critical Win32 gotchas** — Alt key detection, RegisterHotKey failures, WPF/WinForms type conflicts |
| [[debug-log-strategy]] | 🐛 Temporary debug log approach for persistent bugs |

---

## 🔍 Audits

| Page | Description |
|------|-------------|
| [[block-audit-summary]] | Summary of all 21 block audits |
| [[input-safety-audit]] | MaxLength and input validation audit |
| [[trigger-sidebar-audit]] | Trigger sidebar UI audit |

---

## 🧠 Agent Intelligence System

### Core Files

| File | Description |
|------|-------------|
| [AGENTS.md](file:///C:/Users/Maaz/Documents/New%20folder/.agents/AGENTS.md) | 🤖 **Project context** — tech stack, build commands, workflow rules |
| [SOUL.md](file:///C:/Users/Maaz/Documents/New%20folder/Obsidian%20Vault/core/SOUL.md) | 🧬 **Agent identity** — values, decision framework, hard boundaries |
| [DECISIONS.md](file:///C:/Users/Maaz/Documents/New%20folder/Obsidian%20Vault/core/DECISIONS.md) | 📋 **Agent Decision Records (AgDR)** — architectural audit trail |
| [GOTCHAS.md](file:///C:/Users/Maaz/Documents/New%20folder/Obsidian%20Vault/core/GOTCHAS.md) | ⚠️ **Common agent traps** — 5 critical project-specific gotchas |
| [SCHEMA.md](file:///C:/Users/Maaz/Documents/New%20folder/Obsidian%20Vault/core/SCHEMA.md) | 📐 **Wiki maintenance rules** — directory layout, tag taxonomy, page format |

### Skills Library

| File | Description |
|------|-------------|
| [skills-index.skill.md](file:///C:/Users/Maaz/Documents/New%20folder/Obsidian%20Vault/core/skills/skills-index.skill.md) | 🗂️ **Level-0 discovery manifest** — read FIRST to know what skills exist |
| [multi-agent-system.skill.md](file:///C:/Users/Maaz/Documents/New%20folder/Obsidian%20Vault/core/skills/multi-agent-system.skill.md) | Parallel agents, job boards, verifier agents |
| [obsidian-markdown.skill.md](file:///C:/Users/Maaz/Documents/New%20folder/Obsidian%20Vault/core/skills/obsidian-markdown.skill.md) | Obsidian-flavored Markdown — wikilinks, callouts, frontmatter |
| [wiki-maintenance.skill.md](file:///C:/Users/Maaz/Documents/New%20folder/Obsidian%20Vault/core/skills/wiki-maintenance.skill.md) | Ingest/query/update/lint wiki |
| [token-optimization.skill.md](file:///C:/Users/Maaz/Documents/New%20folder/Obsidian%20Vault/core/skills/token-optimization.skill.md) | Context budget management |
| [context-compression.skill.md](file:///C:/Users/Maaz/Documents/New%20folder/Obsidian%20Vault/core/skills/context-compression.skill.md) | Long-session recovery — externalize state, enable handoffs |
| [bug-fixing.skill.md](file:///C:/Users/Maaz/Documents/New%20folder/Obsidian%20Vault/core/skills/bug-fixing.skill.md) | Verified fix workflow |
| [bug-triage.skill.md](file:///C:/Users/Maaz/Documents/New%20folder/Obsidian%20Vault/core/skills/bug-triage.skill.md) | Bug triage and prioritization |
| [code-review.skill.md](file:///C:/Users/Maaz/Documents/New%20folder/Obsidian%20Vault/core/skills/code-review.skill.md) | C#/WPF/AHK code review checklist |
| [ahk-scripting.skill.md](file:///C:/Users/Maaz/Documents/New%20folder/Obsidian%20Vault/core/skills/ahk-scripting.skill.md) | AutoHotkey v2 syntax and patterns |
| [wpf-patterns.skill.md](file:///C:/Users/Maaz/Documents/New%20folder/Obsidian%20Vault/core/skills/wpf-patterns.skill.md) | WPF MVVM patterns, MacroEditorViewModel map |
| [spec-writing.skill.md](file:///C:/Users/Maaz/Documents/New%20folder/Obsidian%20Vault/core/skills/spec-writing.skill.md) | SPEC-first development |
| [web-research.skill.md](file:///C:/Users/Maaz/Documents/New%20folder/Obsidian%20Vault/core/skills/web-research.skill.md) | Web research strategy |
| [qa-testing.skill.md](file:///C:/Users/Maaz/Documents/New%20folder/Obsidian%20Vault/core/skills/qa-testing.skill.md) | 70-test QA checklist |
| [release-packaging.skill.md](file:///C:/Users/Maaz/Documents/New%20folder/Obsidian%20Vault/core/skills/release-packaging.skill.md) | Full release pipeline |
| [sprint-pipeline.skill.md](file:///C:/Users/Maaz/Documents/New%20folder/Obsidian%20Vault/core/skills/sprint-pipeline.skill.md) | Sprint planning and task pipeline |
| [targeted-verify.skill.md](file:///C:/Users/Maaz/Documents/New%20folder/Obsidian%20Vault/core/skills/targeted-verify.skill.md) | Targeted verification workflow |

---

## 📄 Meta

| Page | Description |
|------|-------------|
| [[log]] | 📝 Append-only change history for all wiki modifications |

---

*Last updated: 2026-07-15 — Synced index with actual files. Removed 8 dead links (testing-checklist, monetization-plan, website-improvement-ideas, github-setup, branching-strategy, website-content-rules, whatsnew-update-guide, ahk-uia-inspector-upgrade, app-screenshots). Added 10 missing pages. Added Audits section.*
