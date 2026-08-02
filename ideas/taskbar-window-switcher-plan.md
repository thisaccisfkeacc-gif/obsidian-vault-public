---
tags: [plan, feature, taskbar, switcher, UI, engine]
date: 2026-08-02
status: planned
---

# 🚀 Feature Plan: Smart Taskbar Window Switcher

> **Goal**: A dedicated built-in tool that allows users to switch between actively running taskbar windows using number keys with a customizable prefix (e.g. `Alt + 1..9`), smartly skipping closed/unopened pinned shortcuts.

---

## 🎯 Architectural Overview

* **Placement**: Default Section on the Dashboard as a dedicated built-in feature card.
* **Core Logic**: `JSON Config` → `ScriptCompilerService.cs` → `.ahk` → `PowerX_Engine.exe`.
* **Key Behavior**: Cycles only open/active taskbar windows (unless user explicitly toggles on closed pinned app launching).

---

## 📅 Implementation Roadmap (3 Phases)

### 🎨 Phase 1: UI Prototype & Visual Mockup
* [ ] **Card Component Creation**: Design a sleek "Smart Taskbar Switcher" card in the Default section.
* [ ] **Master Toggle**: ON/OFF switch for instant activation.
* [ ] **Prefix Key Dropdown**: Selectable modifier/leader key (`Alt`, `Win`, `Ctrl`, `Caps Lock`, `Tilde ~`, `Tab`).
* [ ] **Behavior Option**: Toggle for *"Skip Unopened Pinned Apps"* (ON by default).
* [ ] **Live Preview Badge**: Displays active hotkey range preview (e.g. `Alt + 1` ... `Alt + 9`).
* [ ] **Visual Polish Audit**: Ensure theme compliance, dark mode harmony, and crisp typography.

---

### ⚡ Phase 2: Engine Wiring & Backend Logic
* [ ] **App State & Storage**: Add configuration flags (`IsTaskbarSwitcherEnabled`, `TaskbarPrefixKey`, `SkipClosedApps`) to app configuration / DB.
* [ ] **ViewModel Integration**: Bind UI inputs in `DefaultSectionViewModel` to persistent settings.
* [ ] **AHK Script Compiler**: Update `ScriptCompilerService.cs` to generate taskbar window enumeration and switching code.
* [ ] **Window Position Matching**: Implement logic to target open window handles based on left-to-right taskbar order.

---

### 🛡️ Phase 3: Quality Audit, Edge Cases & Bug Scanning
* [ ] **Edge Case Testing**:
  * Multi-window apps (e.g., 3 Chrome windows, 2 Explorer windows).
  * Multi-monitor & Virtual Desktop behavior.
  * Dynamically pinning/unpinning apps while macros are active.
* [ ] **Hotkey Safety Verification**: Ensure chosen prefix does not intercept critical OS shortcuts unexpectedly.
* [ ] **Build & Code Review**: Run clean `dotnet build` verification and perform code standards audit.

---
