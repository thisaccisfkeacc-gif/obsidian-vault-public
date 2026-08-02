# 🎧 Voice Note Feedback & Task Breakdown — Part 2

**Source File:** `New recording 18.m4a`  
**Location:** `C:\Users\Maaz\Downloads\Audios\New recording 18.m4a`  
**Duration:** ~2 minutes 30 seconds  
**Date:** July 28, 2026  

---

## 🎯 Executive Summary (Core Takeaways)

This testing voice note session focuses on **Account Trial UI accuracy, Tray Execution Window Focus, and App Scope List UX**:

* 📊 **Trial Progress Bar Calculation:** In the Account section, trial status displays "1 day remaining", but the progress bar fills only ~20% instead of ~95%–99%. The percentage calculation needs fixing. Without percentage number shown []()
* 🎯 **Tray Menu Focus Stealing:** Executing a macro from the System Tray context menu fails to send keypresses/text because clicking the tray menu steals focus from the target window (e.g., Notepad). Need focus restoration before execution.
* 🏷️ **App Scope Extension Cleanup:** App scope rules display raw extensions like `notepad.exe,chrome.exe`. Strip `.exe` and add clean comma spacing (`Notepad, Chrome`).
* 🔄 **Auto-Revert Empty Scope to Global:** Deleting the last app from `Include Only` or `Exclude Only` list should automatically revert the scope mode back to `Active Everywhere` (Global).

---

## 🐛 Bugs & Issues

### 1. Account Section
* [x] **Trial Period Progress Bar Calculation Bug:** When 1 day remains in the trial period, the progress bar shows only ~20% progress instead of ~95%–99% filled. Fix the progress bar percentage calculation formula in the Account View. But make sure that we don't want the percentage number to be shown. *(Fixed — updated math to 14-day base `14 - TrialDaysRemaining` with `Maximum="14"` in `SettingsDashboardViewModel.cs` and `SettingsDashboardView.xaml`)*

### 2. System Tray Macro Execution
* [x] **Tray Context Menu Focus Stealing:** Triggering/executing a macro directly from the System Tray context menu fails to paste text or execute actions on the target application (e.g., Notepad).  
  * **Root Cause:** Clicking the system tray context menu causes the active window focus to shift away from Notepad to the tray menu.  
  * **Fix:** Store/restore active window handle (`HWND`) before executing tray-triggered macros. *(Fixed — captured `GetForegroundWindow()` before menu popup, restored focus via `SetForegroundWindow` in `TrayIconManager.cs`)*

---

## 💡 Improvements & Enhancements

### 1. App Scoping UX & Formatting
* [x] **Strip `.exe` and Format App Names:** On macro cards, app scope lists display raw strings like `notepad.exe,chrome.exe`. Remove `.exe` extensions and add proper spacing after commas (`Notepad, Chrome`). *(Fixed — added `ScopeDisplayText` property on `ActionItem` & `SnippetItem`, updated card XAML bindings)*
* [x] **Auto-Revert Empty Scope List to Global:** Clearing/deleting the last app from an `Include Only` or `Exclude Only` scope list should automatically reset the App Scope mode back to `Active Everywhere` (Global). *(Verified — auto-revert logic active in `ScriptLibraryViewModel` and `TextSnippetsViewModel`)*

---

---

## ✨ New Feature Ideas
* **Hotkey Recording Conflict Guard:** Prevent hotkey recording conflicts when assigning macro triggers.


