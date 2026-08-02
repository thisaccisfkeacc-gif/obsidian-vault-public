# 🤖 PowerX Keys — Delegation Task: Codebase Feature Scan for `core.txt`

> **SOP Reference**: Delegation Workflow (`/delegate`)  
> **Scanner Agent**: Kiro / Deep Scan Agent  
> **Reviewer & Approver**: Antigravity AI  
> **App Context**: PowerX Keys V2 — Windows Desktop Automation App (C# WPF / .NET 10)

---

## 📌 DELEGATION TASK FOR SCANNER AGENT

> **ATTENTION SCANNER AGENT**: Your task is to perform a targeted audit of the PowerX Keys codebase to find all feature capabilities, trigger modes, action types, and settings that belong in `core.txt`.  
> 
> **Do NOT edit any code files or prompt files directly.** Prepare a concise, structured research report for **Antigravity AI** to review, select, and approve.

---

## 🔍 Scan Locations & Directives

Please scan the following authoritative source files:
1. `PowerX.Core/Models/MacroItem.cs` — Scan all `MacroStepType` enum values and step properties.
2. `PowerX.Core/Models/AppConfig.cs` — Scan all user settings, engine toggles, and performance modes.
3. `PowerX.Services/Services/ScriptCompilerService.cs` — Scan all compiled macro capabilities, AHK v2 header configurations, and execution options.
4. `Obsidian Vault/KNOWLEDGE.md` — Scan architecture decisions and feature specs.

---

## 📥 Required Report Structure for Antigravity Review

Please structure your report into the following sections:

### Section 1: Complete List of All 9 Trigger Modes
List every trigger mode string found in `MacroItem.cs` / `AppConfig.cs` along with a 1-line description.

### Section 2: Complete List of All Action Types (0 to 41)
List all supported macro step action types (Keyboard, Mouse, Delay, Text, ImageSearch, PixelSearch, WindowAction, Popup, Notification, UIElement, LogicIf, Group, Loop, WaitUntil, etc.).

### Section 3: Engine Features & Settings Toggles
List all engine toggles (Turbo Engine Mode, Mouse Physics, Global Humanization, Conflict Detection, Performance Mode, UI Automation Fallback, Custom Overlays, etc.).

### Section 4: Recommended Additions for `core.txt`
Provide a bulleted list of missing feature keywords to add to `core.txt` to guarantee 100% feature awareness.
