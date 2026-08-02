# ⚡ PowerX Keys — AI Macro Builder Agent Master Architecture & Brainstorming Spec

> **SOP Reference**: Multi-AI Collaboration & Brainstorming Protocol (`/factory`)  
> **Target Audience**: AI Agents, LLM Architects, Compiler Engineers, UX Designers  
> **App Context**: PowerX Keys V2 — Windows Desktop Automation App (C# WPF / .NET 10)

---

## 📌 IMPORTANT INSTRUCTIONS FOR REVIEWING AI AGENTS

> **ATTENTION AGENTS**: You must **thoroughly read and digest** the entire context, architecture, constraints, and workflow outlined in this document before providing your answers.  
> 
> Do not provide surface-level or generic responses. We require **deep, actionable technical recommendations, prompt engineering strategies, guardrail designs, and UX micro-interaction ideas** to make the PowerX AI Macro Builder the most reliable and intelligent desktop automation agent in existence.

---

## 🏛️ Executive Summary & Core Product Vision

**PowerX Keys V2** is a high-performance Windows desktop macro automation app. It allows users to bind complex automation workflows (mouse movements, keyboard clicks, text expansion, pixel color detection, image searching, app launching) to hotkeys.

### 🌟 The AI Macro Builder Mission
The **AI Macro Builder** is an intelligent agent within PowerX Keys. When a user asks: *"Build me a macro that auto-clicks every 2 seconds when I press F8, and stops when I press F8 again"*, the AI Macro Builder must:
1. Understand the natural language intent.
2. Generate syntactically flawless **AutoHotkey v2** code and structured JSON configuration.
3. Provide one-click injection into the app engine.
4. Offer intelligent refinement chips (e.g. *"Make faster"*, *"Add sound notification"*, *"Add error handling"*).

---

## ⚙️ Technical Architecture & The Golden Rule

```
Natural Language Prompt 
  ➔ AI Macro Builder Agent 
    ➔ Validated JSON / AHK v2 Code 
      ➔ ScriptCompilerService.cs 
        ➔ Compiled .ahk Script 
          ➔ PowerX_Engine.exe (White-labeled AHK v2 Engine)
```

### 🔒 Core Technical Constraints
1. **Target Syntax**: Strictly **AutoHotkey v2** (AutoHotkey v1 syntax is DEPRECATED and strictly forbidden).
2. **Mandatory Script Headers**:
   ```autohotkey
   #Requires AutoHotkey v2.0
   #SingleInstance Force
   ```
3. **Supported Macro Capabilities**:
   * Mouse Clicks (Left, Right, Middle, Double-click, Drag, Relative & Absolute Coordinates)
   * Key Combinations & Key Sends (`SendInput`, `{Ctrl}`, `{Alt}`, `{Shift}`, `{Win}`)
   * Image Searching (`ImageSearch &OutputX, &OutputY, ...`)
   * Pixel Detection (`PixelGetColor`, color tolerance matching)
   * Loops & Timers (`SetTimer`, `Loop`, `While`)
   * Toggle Hotkeys (State-based start/stop macros)
   * Application Launchers (`Run("chrome.exe")`, window activation `WinActivate`)

---

## 🎯 Strategic Brainstorming Topics for Reviewing AI Agents

Please structure your response by evaluating and answering the following 5 core topics in detail:

---

### 1. 🤖 Topic 1: System Directives & Prompt Engineering
How should we construct the master system prompt for the AI Macro Builder to ensure 100% syntactic accuracy for AutoHotkey v2?
* What negative prompts or guardrail rules must be embedded to prevent LLMs from hallucinating v1 syntax (e.g. `SetTimer, Label, 1000` instead of `SetTimer () => Function(), 1000`)?
* What few-shot code examples should be included in the system prompt for complex patterns (loops, toggles, image searches)?

---

### 2. 🛡️ Topic 2: Pre-Execution Syntax Validation & Auto-Repair Layer
Before showing the generated code to the user or passing it to `ScriptCompilerService.cs`:
* How can our C# backend run a lightweight parser or regex validator to catch common AHK v2 syntax mistakes (e.g. missing `&` variable references, missing quotes, unclosed loops)?
* How can the agent auto-repair broken code before returning it to the user?

---

### 3. 🧪 Topic 3: Background Verification & Sandbox File Testing
We want to test macro generation in the background:
* When a user requests a macro, how should the backend write the draft script to a temporary sandbox file (e.g. `%TEMP%/PowerX_Test.ahk`) and run a silent syntax check (`PowerX_Engine.exe /validate script.ahk`) to verify it compiles cleanly before confirming to the user?
* How should error feedback from the engine be fed back to the AI for instant self-correction?

---

### 4. 🎨 Topic 4: UX Micro-Interactions & Refinement Loops
What UI features and micro-interactions will make the AI Builder feel magical and fast?
* **Interactive Refinement Chips**: What one-click action chips should appear after a macro is generated (e.g., *"⚡ Add Toggle Key"*, *"⏱️ Adjust Speed"*, *"🔊 Add Beep Sound"*, *"🛡️ Add Safety Stop"*)?
* **Progressive Preview**: How should code and step-by-step macro cards be displayed?

---

### 5. 🧩 Topic 5: Handling Complex Multi-Step Automation Workflows
When a user asks for a complex multi-step workflow (e.g., *"Open Notepad, type Hello, wait 2 seconds, search for an image on screen, click it, and save the file"*):
* How should the AI decompose complex multi-step requests into modular, reusable macro blocks?
* How can it ensure proper delays and window-existence checks (`WinWaitActive`) so scripts don't fail on slow PCs?

---

## 📥 Required Output Format for Reviewing AI Agents

When answering this spec, please organize your feedback clearly into:
1. **Executive Assessment & Architecture Rating** (1-10 rating of our approach).
2. **Detailed Responses for Topics 1 through 5**.
3. **Sample Master System Prompt** for AutoHotkey v2 macro generation.
4. **Top 5 Actionable Recommendations** for immediate implementation.
