---
type: lesson
status: active
summary: Living repository of solved bugs, root causes, and verified fixes for PowerX Keys.
last_updated: 2026-07-28
---

# 💡 PowerX Keys — Solved Lessons & Bug Fix Memory

This file records verified solutions to non-obvious bugs, crashes, and performance issues. Agents should check this file when diagnosing recurring issues.

---

## 🐛 Bug Fix Memory

### 1. Desktop Refresh Flickering on App Launch / Restore
* **Problem**: Restoring an existing instance of PowerX Keys broadcasted a window message causing Windows Explorer desktop icons to flicker and redraw.
* **Root Cause**: `HWND_BROADCAST` was being used in `App.xaml.cs` along with redundant `SHChangeNotify(SHCNE_ASSOCCHANGED)` calls.
* **Verified Fix**:
  1. Replaced `HWND_BROADCAST` with single-instance HWND file handoff (`%TEMP%\powerx_hwnd.txt`).
  2. Checked registry in `FileAssociationService.cs` before registering `.pxmacro` extensions to avoid unnecessary `SHChangeNotify` calls on launch.

---

### 2. Settings Dashboard Theme Circle Swatches
* **Problem**: Light mode theme selection circle swatches in Settings view appeared dark and unreadable.
* **Root Cause**: Hardcoded dark background fills in `SettingsDashboardView.xaml`.
* **Verified Fix**: Updated `SettingsDashboardView.xaml` to use `#F8F9FA` for light mode swatch fill and `#1E1F29` for dark mode fill.
