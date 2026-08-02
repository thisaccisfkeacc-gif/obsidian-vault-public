# Voice Note Feedback 2 — Verification Report

## Bug 1: Trial Progress Bar Calculation
**Status: VERIFIED — Needs Fix**

- **ProgressBar** at `SettingsDashboardView.xaml:2661` has `Maximum="30"` and `Value="{Binding TrialDaysRemaining}"`
- `TrialDaysRemaining` (`SettingsDashboardViewModel.Commands.cs:500-512`) returns raw days **remaining** (e.g., 1)
- **Bug:** 1 day remaining → Value=1, Max=30 → bar fills **~3.3%** instead of ~97%
- **Fix needed:** Change Value binding to `30 - TrialDaysRemaining` or create a new computed property
- **Percentage number:** Already NOT shown — no percentage label exists. Requirement met.

## Bug 2: Tray Menu Focus Stealing
**Status: VERIFIED — Needs Fix**

- `ExecuteMacroFromTray` (`TrayIconManager.cs:365-375`) calls `MacroExecutionService.ExecuteMacroAsync(macro)` directly
- **No HWND save/restore** — focus is not preserved before execution
- `SetForegroundWindow` exists at `TrayIconManager.cs:127` but is only used for main window restore, not for target window focus
- **Fix needed:** Save foreground HWND before tray click, restore it before executing macro

## Improvement 1: Strip .exe & Format App Names
**Status: VERIFIED — Needs Fix**

- `TargetAppList` (`AppConfig.cs:730`) splits raw string by commas and trims — no `.exe` stripping or formatting
- `TargetApp` display in `CustomActionCard.xaml:1526` shows raw binding `{Binding}` directly — shows `notepad.exe,chrome.exe`
- **Fix needed:** In `TargetAppList` getter, strip `.exe` and format cleanly (e.g., `Notepad, Chrome`)

## Improvement 2: Auto-Revert Empty Scope to Global
**Status: VERIFIED — Needs Fix**

- `ExecuteRemoveTargetApp` (`ScriptLibraryViewModel.Commands.cs:726-754`) removes the app from list but **never checks** if the list becomes empty
- **Fix needed:** After removal, if `TargetAppsCount == 0`, set `ScopeMode = AppScopeMode.Global`

---

**Summary:** All 4 claims are true. All 4 need code changes.
