---
tags: [feature, ui, polish]
date: 2026-07-15
status: completed
---

# Error Handling Cleanup — No Ugly Popups

**Goal:** The user should NEVER see a raw/ugly error dialog. Every error, warning, or unexpected behavior gets shown in the app's dark theme via `DarkMessageBoxWindow.Show()`.

**Status: ✅ COMPLETED**

---

## What Was Fixed

### 1. AHK Engine Runtime Errors — ✅ Fixed
- Added `/ErrorStdOut` flag to engine launch arguments
- Added `RedirectStandardError = true` to process StartInfo
- On engine exit with non-zero code: reads stderr, logs it, shows `DarkMessageBoxWindow` with friendly message
- **File:** `Managers/ScriptManager.cs`

### 2. Stray `MessageBox.Show()` Calls — ✅ All 5 Replaced

| # | File | What it showed | Fix |
|---|------|---------------|-----|
| 1 | `Views/CaptureLibraryWindow.xaml.cs` | Cleanup confirmation | → `DarkMessageBoxWindow.Show` |
| 2 | `Views/CaptureLibraryWindow.xaml.cs` | "Image file not found" | → `DarkMessageBoxWindow.Show` |
| 3 | `Services/UIElementCaptureService.cs` | "Capture timed out" | → `Views.DarkMessageBoxWindow.Show` |
| 4 | `ViewModels/MacroEditorViewModel.Capture.cs` | "Failed to run preview: {ex}" | → Friendly message, no stack trace |
| 5 | `ViewModels/ScriptLibraryViewModel.Commands.cs` | "UI Element capture failed: {ex}" | → Friendly message, no stack trace |

### 3. Crash Dialog — ✅ Simplified
- **Before:** "A critical error occurred: Value cannot be null (Parameter 'source'). Please check crash.txt for details."
- **After:** "Something went wrong and the app needs to restart."
- Technical details still saved silently to `crash.txt` for debugging
- **File:** `App.xaml.cs`

---

## Notes
- The `AppDomain.UnhandledException` handler doesn't show a dialog (just writes crash.txt and shuts down) — correct behavior
- `TaskScheduler.UnobservedTaskException` is silently observed and logged — correct behavior
- No user-facing error will ever show technical text, stack traces, or file paths
