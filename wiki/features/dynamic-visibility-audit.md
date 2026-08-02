---
tags: [feature, ui, audit]
date: 2026-07-06
sources:
  - MainWindow.xaml
  - MacroEditorView.xaml
  - SettingsDashboardView.xaml
  - CustomActionCard.xaml
  - MacroStepCard.xaml
  - ScriptLibraryView.xaml
  - TextSnippetsView.xaml
status: active
---

# 🔍 Dynamic Visibility Audit — Options That Appear/Disappear

**Summary:** This is a complete list of all options, buttons, panels, and interactive elements in the app that **dynamically appear or disappear** based on some condition. The goal is to review each one and decide whether to change it from "hidden until condition" → "always visible but grayed out until condition" for better UX.

> **Why grayed out is better:** Users can see the option exists even before the condition is met. This reduces confusion and increases discoverability. The option is visible but disabled, and once the condition is met, it becomes active.

---

## ⭐ STRONG CANDIDATES (Recommended for Gray-Out)

These are the best candidates because users **can't discover** these features unless the condition happens to be active. Graying them out would let users know these options exist.

---

### 1. 🖼️ Screen Event Capture — Image/Pixel/UIElement Mode Panels
**Where:** Custom Action Card (hotkey card settings)
**Current behavior:** When trigger mode is "Screen Event," three sub-panels (Image mode, Pixel mode, UI Element mode) swap in/out based on the selected detection mode. Only one is visible at a time, the other two are fully hidden.
**Condition:** `ScreenEventDetectMode == Image/Pixel/UIElement`
**Why gray out:** A user in Pixel mode has no idea the Image mode or UI Element mode panels exist. Graying them out (with a label like "Switch to Image mode to use this") makes all options visible.

---

### 2. 📸 "Capture Image" / "Capture Pixel" / "Capture UI Element" Buttons
**Where:** Custom Action Card
**Current behavior:** The "Capture" button appears when nothing is captured yet. Once something is captured, it swaps to a "captured state" panel showing the result. The capture button disappears.
**Condition:** `HasTriggerImage/HasTriggerPixel/HasTriggerUIElement == False` shows button, `True` hides it
**Why gray out:** After capturing, the user loses the ability to re-capture unless they clear first. Keeping a (dimmed) re-capture option visible could help.

---

### 3. ⏱️ Custom Poll Interval Input
**Where:** Custom Action Card (Screen Event settings — appears in Image, Pixel, AND UI Element modes)
**Current behavior:** A text input for custom polling interval only appears when the poll rate dropdown is set to "Custom."
**Condition:** `IsScreenEventPollCustom == True`
**Why gray out:** Users don't know they can set a custom poll interval unless they pick "Custom" from the dropdown first. Showing it grayed out with "Select 'Custom' poll rate to use" makes it discoverable.

---

### 4. ❄️ Cooldown Input Field
**Where:** Custom Action Card (Screen Event settings — all three modes)
**Current behavior:** A cooldown timer input only appears when cooldown is enabled.
**Condition:** `IsScreenEventCooldownVisible == True`
**Why gray out:** Same discoverability issue — users don't know cooldown is available unless they toggle it on.

---

### 5. 🔔 Notification Sub-Options (Tooltip & Sound)
**Where:** Custom Action Card (Screen Event settings — all three modes)
**Current behavior:** When "Notify on Found" is turned on, extra options for tooltip and sound notification appear below it.
**Condition:** `ScreenEventNotifyOnFound == True`
**Why gray out:** Good candidate — shows users they can customize the notification style even before enabling it.

---

### 6. 🎯 Target Window / Target App Section
**Where:** Custom Action Card + Text Snippets View
**Current behavior:** The "Capture Target Window" button and the target app pills list are hidden when Scope is set to "Global." They only appear for "Include" or "Exclude" scope modes.
**Condition:** `ScopeMode != Global`
**Why gray out:** Strong candidate — users on Global scope have no idea they can target specific apps. Graying it out with "Switch scope to Include/Exclude to target specific apps" is much better.

---

### 7. 📦 "Move to Profile" Section
**Where:** Custom Action Card
**Current behavior:** The option to move a hotkey to a different profile only appears when the user has created custom profiles. If there are no custom profiles, this option is invisible.
**Condition:** `HasCustomProfiles == True`
**Why gray out:** Very strong candidate — users don't know they can organize hotkeys across profiles. Graying it out with "Create a custom profile first" teaches them profiles exist.

---

### 8. 🎹 Hotkey Assignment Section (Bottom of Action Card)
**Where:** Custom Action Card
**Current behavior:** The entire bottom section for assigning a keyboard shortcut is hidden when the trigger mode is Screen Event, Schedule, or Mobile Remote.
**Condition:** `TriggerMode != ScreenEvent/Schedule/MobileRemote`
**Why gray out:** Users switching trigger modes might wonder where the hotkey section went. Graying it out with "Not available for this trigger mode" is clearer.

---

### 9. 🤖 Humanize Level Settings
**Where:** Macro Editor View (Playback settings)
**Current behavior:** The "Human Feel" combo box (humanization intensity) only appears when Global Humanization is enabled.
**Condition:** `GlobalHumanizationEnabled == True`
**Why gray out:** Users don't know they can fine-tune the humanization level. Graying it out prompts them to enable humanization first.

---

### 10. ⏳ Auto-Delay Settings
**Where:** Macro Editor View (Playback settings)
**Current behavior:** The delay milliseconds input only appears when auto-delay is enabled for the current macro.
**Condition:** `CurrentMacro.AutoDelayEnabled == True`
**Why gray out:** Similar to humanization — the setting is invisible until auto-delay is toggled on.

---

### 11. 👁️ "Preview Step" in Right-Click Menu
**Where:** Macro Editor View (timeline context menu)
**Current behavior:** The "Preview Step" option in the right-click menu only appears for certain step types (ImageSearch with image, PixelSearch with valid pixel, UIElement with type, WindowAction with title, SystemSound).
**Condition:** Complex multi-condition per step type
**Why gray out:** Users right-clicking on a Mouse Click step have no idea "Preview" exists for other step types. Graying it out with "Preview not available for this step type" teaches them it exists.

---

### 12. 🗑️ Delete Profile Button
**Where:** Script Library View (sidebar)
**Current behavior:** The delete button only appears on hover AND only for custom profiles. Built-in profiles (Default, Custom Actions, Macro Bindings) never show a delete option.
**Condition:** Profile is custom (not system) + mouse hover
**Why gray out:** Minor candidate — could show grayed out for system profiles with "Built-in profiles can't be deleted."

---

### 13. 📂 "MY PROFILES" Section in Sidebar
**Where:** Main Window (left sidebar)
**Current behavior:** The entire "My Profiles" section with custom profiles is hidden when the user has zero custom profiles.
**Condition:** `HasCustomProfiles == True`
**Why gray out:** Good candidate — showing an empty "My Profiles" section with a "Create Profile" prompt teaches users profiles exist.

---

### 14. 📱 Mobile Remote QR Code & Connection Details
**Where:** Settings Dashboard (Engine section)
**Current behavior:** The QR code and connection details panel only appears when Remote Server is enabled in settings.
**Condition:** `RemoteServerEnabled == True`
**Why gray out:** Users don't know the remote feature has a QR code. Showing it grayed out with "Enable Remote Server to see connection details" improves discoverability.

---

### 15. 🏭 Factory Reset Options Panel
**Where:** Settings Dashboard
**Current behavior:** The reset options (what to reset) only appear when the user clicks to expand them.
**Condition:** `IsResetOptionsVisible == True`
**Why gray out:** Minor — this is a standard expand/collapse pattern. Could go either way.

---

## 🟡 MODERATE CANDIDATES (Could Go Either Way)

These are valid options but the benefit is less clear. Use your judgment.

---

### 16. ⚠️ Humanization Warning Banner
**Where:** Macro Editor View
**Current behavior:** A warning banner about humanization appears/disappears based on certain conditions.
**Condition:** `IsHumanizationWarningVisible == True`
**Why maybe:** Warnings are typically shown only when relevant. Graying out a warning doesn't make much sense.
**My opinion:** Keep as dynamic.

---

### 17. 🏷️ Hotkey Badge on Macros
**Where:** Main Window (macro list in sidebar)
**Current behavior:** A keyboard shortcut badge (showing the key combo) appears next to each macro only when a hotkey is assigned.
**Condition:** Hotkey is not null/empty
**My opinion:** Keep as dynamic — showing an empty hotkey badge on every macro would be cluttered.

---

### 18. 🔴 "HIGH CPU" Warning Pill
**Where:** Custom Action Card (UI Element mode)
**Current behavior:** A red "HIGH CPU" warning badge appears only when Monitor Mode is set to "Always."
**Condition:** `ScreenEventUIMonitorMode == Always`
**My opinion:** Keep as dynamic — it's a contextual warning, not a feature to discover.

---

### 19. ⚡ Scope Tags (App Name Pills)
**Where:** Custom Action Card
**Current behavior:** Shows app name pills (like "chrome.exe") next to the action card only when scope is Include/Exclude AND target apps are set.
**Condition:** `ScopeMode == Include/Exclude` AND `HasTargetApps == True`
**My opinion:** Keep as dynamic — these are status indicators, not interactive options.

---

### 20. 🔄 "Reset Hotkey" Button
**Where:** Custom Action Card
**Current behavior:** A reset button only appears when the user is currently in the middle of binding a hotkey.
**Condition:** `IsBinding == True`
**My opinion:** Keep as dynamic — it's a contextual action during an active operation.

---

### 21. 📊 Full Changelog History
**Where:** Settings Dashboard (What's New section)
**Current behavior:** Full changelog is collapsed by default, expands when user clicks "Show All."
**Condition:** `IsFullHistoryVisible == True`
**My opinion:** Keep as expand/collapse — this is standard UX.

---

## ❌ NOT CANDIDATES (Should Stay Dynamic)

These should stay as-is because they're **status indicators, loading states, animations, or standard UI patterns**.

| # | Element | Why Keep Dynamic |
|---|---------|-----------------|
| 22 | Engine spinner (loading) | Loading state — not a user option |
| 23 | Purple pulse border (AI building) | Active state indicator |
| 24 | Update notification dot (red dot) | Notification badge — standard pattern |
| 25 | Update popup banner | One-time notification — not an option |
| 26 | Tip Jar popup | Promotional popup — not discoverable option |
| 27 | Analytics popup | Toggle panel opened by button click |
| 28 | AI Assistant panel | Toggle panel opened by button click |
| 29 | AI Live Build badge | Status indicator during operation |
| 30 | Error count badge | Status indicator |
| 31 | Recording floating widget | Active during recording only |
| 32 | Drag-drop indicator line & ghost | Drag UX — must be dynamic |
| 33 | Onboarding guide overlays | First-time experience — must be dynamic |
| 34 | Timeline empty state | Placeholder — must be dynamic |
| 35 | Running indicator dots (green) | Status — must be dynamic |
| 36 | QR loading spinner | Loading state |
| 37 | Remote access status text | Status text |
| 38 | Tunnel connecting overlay | Loading state |
| 39 | MacroStepCard per-type panels | Type-specific config — switching is core UX |
| 40 | Section collapsers (Basic Actions, Logic, Advanced) | Standard folder expand/collapse |
| 41 | Step summary texts | Contextual display per step type |
| 42 | View/Playback settings panels | Standard toggle sections |
| 43 | Settings category panels | Standard tab switching |
| 44 | Color swatch checkmark | Selection indicator |
| 45 | Playback Speed selector | Currently hardcoded Collapsed (unfinished feature) |
| 46 | Smart/Raw View modes | Currently hardcoded Collapsed (unfinished feature) |
| 47 | Timeline Layout cycle button | Currently hardcoded Collapsed (unfinished feature) |

---

## 📋 Quick Reference Summary

| Priority | Count | Description |
|----------|-------|-------------|
| ⭐ Strong candidates | 15 | Should definitely be reviewed for gray-out |
| 🟡 Moderate candidates | 6 | Could go either way |
| ❌ Not candidates | 26 | Should stay dynamic |

---

## Key Files
- [MainWindow.xaml](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/MainWindow.xaml) — 15+ visibility elements
- [MacroEditorView.xaml](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/MacroEditorView.xaml) — 15+ visibility elements
- [SettingsDashboardView.xaml](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/SettingsDashboardView.xaml) — 10+ visibility elements
- [CustomActionCard.xaml](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/CustomActionCard.xaml) — 20+ visibility elements
- [MacroStepCard.xaml](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/MacroStepCard.xaml) — 25+ visibility elements
- [ScriptLibraryView.xaml](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/ScriptLibraryView.xaml) — 5+ visibility elements
- [TextSnippetsView.xaml](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/TextSnippetsView.xaml) — 5+ visibility elements

## Related Pages
- [[settings-dashboard]]
- [[macro-editor]]
- [[script-library]]
- [[capture-library]]
