# 🚀 PowerX Keys — Phase 2 Master Batch Testing Strategy & Handoff Spec

> **Session Handoff Date**: July 31, 2026  
> **Target System**: Phase 2 AI Macro Builder Agent  
> **App Context**: PowerX Keys V2 — Windows Desktop Automation App (C# WPF / .NET 10)  
> **Strategy**: 10 Batches x 10 Prompts (100 Total Real-World Macro Tests)

---

## 📌 COPY-PASTE PROMPT FOR NEW AGENT SESSION

```text
Hello! I am starting a fresh session for Phase 2: AI Macro Builder Batch Testing & Audit. 
Please read PHASE2_BATCH_TESTING_STRATEGY_HANDOFF.md on Desktop to get 100% context on our 10-Batch testing strategy.

Key Highlights of the Strategy:
1. 10 Batches x 10 Prompts (100 Total Realistic Prompts focusing on basic & medium tasks, no over-engineered edge cases).
2. Smart Handling of Missing Real Data (Images, Pixels, UI Elements, File Paths):
   - AI outputs clean placeholders (e.g. "image.png", "C:\path\to\app.exe") AND adds a friendly 1-sentence tip encouraging the user to capture their real image/file path in PowerX Keys.
3. Multi-Agent Pipeline:
   - Agent 1: Prompt Generator (Prepares batch prompts)
   - Agent 2: API Tester (Runs requests with rate-limit delays)
   - Agent 3: Smart Auditor (Pro Model: Inspects JSON schema, Action Types 0-42, nesting 30/31/22, and logs improvements into PHASE2_MACRO_TEST_AUDIT.md).

Please confirm you have read this handoff spec and are ready to execute Batch 1! 🚀
```

---

## 🏛️ Comprehensive 10-Batch Test Roadmap

### 📋 Overview of 10 Test Batches (10 Prompts per Batch):

| Batch # | Focus Area | Key Concepts Tested |
|:---|:---|:---|
| **Batch 1** | Basic Keystrokes & Text Typing | Action 0 (Key), Action 8 (Text), `0|Press|Key+Key` combinations |
| **Batch 2** | Mouse Clicks & Delays | Action 3 (Mouse), Action 7 (Delay), Mouse Physics parameters |
| **Batch 3** | Window Actions & File Launchers | Action 12 (WindowAction), Action 19 (FileLauncher with Path Placeholders + Tips) |
| **Batch 4** | Simple Loops & Key Combinations | Action 31 (Loop), Action 22 (EndTag), single-step combo shortcuts |
| **Batch 5** | ImageSearch & PixelSearch | Action 10 (Image), Action 11 (Pixel with Image Placeholders + Capture Tips) |
| **Batch 6** | UI Automation Elements | Action 23 (UIElement with Selector Placeholders + Capture Tips) |
| **Batch 7** | Conditional Logic Blocks | Action 20 (If), Action 21 (Else), Action 22 (Universal EndTag) |
| **Batch 8** | Notifications, Popups & Sounds | Action 14 (Popup), Action 15 (Notification), Action 16 (SystemSound) |
| **Batch 9** | Multi-Action Practical Workflows | Real-world daily tasks (Browser automation, Gaming macro, Form filling) |
| **Batch 10** | Macro Chaining & Variables | Action 41 (CallMacro), Action 42 (SetVariable), Emergency Kill-Switch |

---

## 💡 Smart Real-Data Placeholder & Tip Rules

When a macro prompt requests actions that require local user data (Images, Pixels, UI Elements, File Paths) without specifying exact paths:

1. **Clean Placeholder Output**:
   * ImageSearch (10): Use `"target_image.png"`
   * PixelSearch (11): Use `"11|#FF0000|100|200|50|50"`
   * UIElement (23): Use `"23|Target Button|Click|Window Title"`
   * FileLauncher (19): Use `"19|C:\\Program Files\\App\\app.exe"`

2. **Friendly 1-Sentence User Guidance Tip**:
   * Above the JSON block, add a helpful tip:  
     *e.g. "Tip: Replace 'target_image.png' with your captured image filename in PowerX Keys Image Manager!"*

---

## 🛠️ Multi-Agent Role Execution Pipeline

```text
[ Batch Prompt Generator ] ➔ [ API Tester (Rate-Limit Delays) ] ➔ [ Smart Auditor Agent (Pro Model) ]
                                                                           │
                                                                           ▼
                                                             PHASE2_MACRO_TEST_AUDIT.md
```

1. **Rate-Limit Safe**: Pauses 2–3 seconds between prompt calls so free API keys never hit RPM bans.
2. **Silent Observation Logging**: All audit results, schema passes, and improvement notes are recorded in `PHASE2_MACRO_TEST_AUDIT.md`.
3. **User Approval**: Final improvements are reviewed with the user before code implementation.
