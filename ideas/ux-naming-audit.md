---
tags: [wiki, ux, naming, collaborative]
date: 2026-06-18
status: active
contributors: [Antigravity]
---

# 🔍 PowerX Keys — UX Naming & Confusion Audit

> **Collaborative document.** Verified against actual XAML + ViewModel source.
> 3 rounds of filtering — only the genuinely worth-fixing issues remain.
> Last updated: 2026-06-18 | Final count: 15 real issues

---

## 📋 How to Contribute (For Other Agents)
- ✅ **Add a finding** — new row + exact file name
- ✅ **Suggest alternative** — update the Fix column
- ✅ **Mark fixed** — add ✅ FIXED + date to Status column
- ❌ **Don't re-add** anything in the Removed sections below

---

## ❌ Removed — False Positives (Never Existed in UI)

| Item | Reason |
|---|---|
| "Saved Macros" sidebar | Not found in any XAML |
| "Smart Wait" | Internal C# only — not shown to user |
| "Text Expander" vs "Text Snippets" | UI is consistent — "Text Snippets" everywhere |
| "WaitForKey" label | Tag attribute only — card says "Wait for Key Press" |
| "User Input" confusion | Card correctly says "Ask User Input" |
| "HOW IT RUNS" | XAML comment only — not rendered |

---

## ❌ Removed — Real But Not Worth Fixing

| Item | Why Cut |
|---|---|
| "Record All Movement" | Clear tooltip explains it |
| "Group Folder" | "Folder" = container is universally understood |
| "Mouse Trace" | Accepted term in automation tools |
| "Quick Actions" confusion | Empty state hint explains it |
| "Smart View" | Tooltip says exactly what it does |
| "Auto-insert Event Delays" | Tooltip covers it |
| "Performance Mode (Low-End PC)" | Parenthetical actually helps |
| "Wait Until" | Self-describing — you wait UNTIL a condition |
| "Extreme Paste Speed" | "Extreme" is colloquially understood, not confusing |
| "Fill-in-the-blank" | Charming and intuitive in snippets context |
| Toast: "assign a Target File path" | Error toast — transient, low impact |
| Toast: "assign the Keystroke to simulate" | Same — transient, low impact |
| "APP SWITCH SPEED" | Enough surrounding context in the UI |
| "Performance Mode" overlap with "Zero-Latency" | Fixing #3 already resolves this |

---

## 🔴 High — Fix These First

| # | File | Current Label | Problem | Suggested Fix | Status |
|---|---|---|---|---|---|
| 1 | `CustomActionCard.xaml` | **"Screen Event"** | Sounds like a Windows OS event, not image detection | "When Image Appears" | 🔲 Open |
| 2 | `SettingsDashboardView.xaml` | **"VIP Admin Mode"** | Reads like a joke/Easter egg. Subtitle correctly says "Restarts with UAC elevation" — title should match | "Run as Administrator" | 🔲 Open |
| 3 | `SettingsDashboardView.xaml` | **"Zero-Latency Override"** | Implies speed. Subtitle says "Forces Windows to prioritize macros" — that's High Priority, not latency | "Turbo Engine Mode" — implemented 2026-08-01 (smart boost while running, default OFF) | ✅ Resolved |
| 4 | `MacroEditorOverlays.xaml` | **"TRIGGER TYPE"** | Same concept called "TRIGGER MODE" in MacroEditorView — inconsistent | Standardize to "Trigger Mode" | 🔲 Open |
| 5 | `MacroStepCard.xaml` | **"UI Element"** | Technical WPF jargon — most users won't know what this means | "Click App Button" | 🔲 Open |
| 6 | `MacroStepCard.xaml` | **"Call Macro"** | "Call" is a programming term | "Run Another Macro" | 🔲 Open |
| 7 | `MacroStepCard.xaml` | **"Set Variable"** | Programming jargon | "Save a Value" | 🔲 Open |
| 8 | `SettingsDashboardView.xaml` | **"SMART MODE LOGIC"** | "Smart" explains nothing — it's conditional branching | "Conditional Logic" | 🔲 Open |

---

## 🟡 Medium — Worth Fixing Soon

| # | File | Current Label | Problem | Suggested Fix | Status |
|---|---|---|---|---|---|
| 9 | `MacroEditorView.xaml` | **"HUMAN FEEL"** section + **"Humanize Timing"** checkbox | Two different names for the same concept on the same screen | Use "Humanize Timing" for both | 🔲 Open |
| 10 | `SettingsDashboardView.xaml` | **"Startup Update Popups"** | "Popups" sounds like ads or spam | "Show Update Alerts on Startup" | 🔲 Open |
| 11 | `MacroEditorView.xaml` | **"IGNORE SHORT PRESSES"** | "Short" is undefined — no threshold shown | "Ignore Quick Taps" + show ms value | 🔲 Open |
| 12 | `MacroEditorOverlays.xaml` | **"Bind to Dashboard Container"** | "Container" is jargon — tooltip explains it but the label doesn't | "Add to My Macros Panel" | 🔲 Open |

---

## 🟢 Minor Polish — Low Effort, Still Real

| # | File | Current Label | Issue | Suggested Fix | Status |
|---|---|---|---|---|---|
| 13 | `ScriptLibraryView.xaml` | **"App / File Launcher"** | Same thing called "Launch File" in MacroEditorView — inconsistent | "Launch File" | 🔲 Open |
| 14 | `SettingsDashboardViewModel.cs` | Toast: **"Mobile Remote failed to start. Port may be in use."** | "Port" is developer jargon | "Phone Remote couldn't start — try restarting the app" | 🔲 Open |
| 15 | `CustomActionCard.xaml` | **"Mobile Remote"** trigger mode | Same name used for the trigger mode AND the settings card — two different things | Rename trigger to "Phone Trigger" | 🔲 Open |

---

## ✅ Well-Named — Do Not Change

| Label | Why |
|---|---|
| "Double Tap" | Crystal clear |
| "Delay" | Simple and universal |
| "Find Image" / "Find Pixel" | Clear and descriptive |
| "Key Down" / "Key Up" | ✅ Being implemented |
| "Factory Reset" | Standard Windows term |
| "Crash Report" | Honest and clear |
| "Ask User Input" | Friendly phrasing |
| "Launch File" | Simple and direct |
| "Wait for Key Press" | Already clear |
| "No macros here yet" | Friendly empty state |
| Tooltip: *"Magically smooths out jagged, robotic mouse movements into a buttery curve."* | Keep — great writing |
| Tooltip: *"Locks your mouse and keyboard while the macro runs. Your emergency stop key still works."* | Keep — reassuring |

---

## 💬 Agent Comments

**Antigravity — 2026-06-18:**
- 3 rounds of filtering: 40 raw → 22 → **15 final genuine issues**
- Key insight: subtitles and tooltips in this app are actually well-written. The real problems are concentrated in step block titles and 4-5 settings labels.
- Toasts are mostly fine — only the "Port" toast was bad enough to keep.
- "Wait Until", "Extreme Paste Speed", and "Fill-in-the-blank" all cut — they hold up under scrutiny.
