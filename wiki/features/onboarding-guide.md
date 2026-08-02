---
tags: [feature, ui, onboarding]
date: 2026-08-01
sources:
  - ViewModels/ScriptLibraryViewModel.cs
  - ViewModels/TextSnippetsViewModel.cs
  - Views/ScriptLibraryView.xaml
  - Views/TextSnippetsView.xaml
  - Models/AppConfig.cs
status: active
---

# Onboarding Guide System

**Summary:** A section-aware card popup system that shows first-time users a 3-step guide when they visit Quick Actions, My Macros, or Text Snippets for the first time.

---

## Overview

The guide appears as a frosted-glass card in the **bottom-right corner** of the screen. It shows automatically on first visit to each section and is permanently dismissed once the user clicks "Got it" on step 3.

- 3 steps per section (dots + step counter in footer)
- Scale + fade entrance animation (0.24s, CubicEase)
- DropShadowEffect for depth
- × button to dismiss early at any step
- "Next →" advances steps, "Got it" on step 3 dismisses and saves

---

## Sections Covered

### ⚡ Quick Actions

| Step | Title | Message |
|------|-------|---------|
| 1 | Welcome to Quick Actions | Quick Actions let you open apps, press key combos, or switch windows instantly. |
| 2 | Add your first action. | Hit + Add Action in the top right. Pick what you want it to do and save. |
| 3 | Assign a hotkey, press Start. | Pick a key combo for your action, then hit Start up top. It runs anywhere on your PC. |

### 🎬 My Macros

| Step | Title | Message |
|------|-------|---------|
| 1 | Welcome to My Macros | This is where your saved macros live. To create one, go to Macro Editor and click the purple Create button. |
| 2 | Assign a hotkey here. | Once saved, your macro appears here. Set a hotkey, pick a trigger mode, and customize the icon. |
| 3 | Press Start and you're done. | Hit Start at the top. Your macro will fire with that hotkey anywhere on your PC. |

> [!IMPORTANT]
> Macros are **created** in the Macro Editor section — NOT in My Macros. My Macros only assigns hotkeys to already-saved macros.

### 📝 Text Snippets

| Step | Title | Message |
|------|-------|---------|
| 1 | Welcome to Text Snippets | Snippets expand short triggers into full text automatically as you type — no hotkey needed. |
| 2 | Create your first snippet. | Click + Add Snippet, set a short trigger like .email, and paste your full text as the output. |
| 3 | Just type the trigger. | Enable the snippet, press Start, and type your trigger anywhere. It expands instantly. |

### 🎬 Macro Editor

| Step | Title | Message |
|------|-------|---------|
| 1 | Welcome to Macro Editor | This is where you build macros. Each macro is a sequence of steps — key presses, clicks, waits, and more. |
| 2 | Add steps or record. | Click + Add Step to build manually, or hit Record to capture your real keyboard and mouse actions automatically. |
| 3 | Save and head to My Macros. | When done, hit Save. Your macro will appear in My Macros where you can assign it a hotkey and run it. |

---

## Architecture

### State Flags (AppConfig.cs)

Four boolean flags persist the "has seen" state per section:

```csharp
public bool HasSeenQuickActionsGuide { get; set; } = false;
public bool HasSeenMacrosGuide { get; set; } = false;
public bool HasSeenSnippetsGuide { get; set; } = false;
public bool HasSeenMacroEditorGuide { get; set; } = false;
```

Saved via `ConfigManager.Save()` on dismiss.

### ViewModel Properties

Both `ScriptLibraryViewModel` and `TextSnippetsViewModel` expose:

```csharp
public bool IsGuideVisible { get; set; }
public int GuideStep { get; set; }        // 0, 1, 2
public string GuideTitle { get; set; }
public string GuideMessage { get; set; }
```

### Commands

| Command | Action |
|---------|--------|
| `NextGuideStepCommand` | Advances step or dismisses on step 2 |
| `DismissGuideCommand` | Hides card, marks as seen, saves config |

### Section Detection (ScriptLibraryViewModel)

The script library is shared between Quick Actions and My Macros. A private field tracks which section is active:

```csharp
private string _currentGuideSection; // "CustomActions" or "MacroBindings"
```

Set in the `CurrentDisplayProfile` setter. `CheckSectionGuide()` is called on profile switch and at constructor time.

---

## Key Files

- [ScriptLibraryViewModel.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/ViewModels/ScriptLibraryViewModel.cs) — Guide logic for Quick Actions + My Macros (`CheckSectionGuide`, `UpdateGuideStep`, `ExecuteDismissGuide`)
- [TextSnippetsViewModel.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/ViewModels/TextSnippetsViewModel.cs) — Guide logic for Text Snippets
- [ScriptLibraryView.xaml](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/Views/ScriptLibraryView.xaml) — Guide card XAML (bottom of file, `QUICK ACTIONS ONBOARDING GUIDE POPUP` comment)
- [TextSnippetsView.xaml](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/Views/TextSnippetsView.xaml) — Guide card XAML (inside root Grid, near end of file)
- [AppConfig.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Core/Models/AppConfig.cs) — `HasSeen*Guide` flags

---

## Related Pages

- [[script-library]]
- [[app-config]]
