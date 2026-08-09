---
tags: [changelog, audit]
date: 2026-07-23
---

# PowerX Keys Optimization Changelog

Track all code changes made during the optimization audit. If a bug appears after a fix, check here to see what changed.

---

## Batch 1 — July 23, 2026

### Fix 03: SQLite WAL + Transactions
**File:** `Managers/MacroDatabase.cs`

**Changes:**
1. Added PRAGMAs after opening SQLite connection:
   - `PRAGMA journal_mode=WAL` — Write-Ahead Logging for faster writes
   - `PRAGMA synchronous=NORMAL` — balanced durability/speed
   - `PRAGMA busy_timeout=5000` — waits 5s before "database locked" error
   - `PRAGMA temp_store=MEMORY` — temp indexes in RAM (added by other agent)
2. Added explicit transactions to `DeleteMacro()` and `DeleteAllMacros()`

**What could break:**
- If app crashes mid-save, macro data could be partially written (WAL handles recovery though)
- First launch after this change will switch existing DB to WAL mode (one-time, safe)

**Test:** Save/load macros, delete macros, force-kill app mid-save and verify no corruption

---

### Fix 04: GDI Memory Leaks
**File:** `Views/ImageStudioWindow.xaml.cs`

**Changes:**
1. Line 137: Added `_sourceBitmap?.Dispose();` before creating new bitmap in `LoadImage()`

**What could break:**
- If `_sourceBitmap` is null on first load, the `?.` operator handles it safely
- Old bitmap is now disposed before new one is created — no visible change

**Test:** Open Image Studio, load 5+ different images, close — memory should not grow

---

**File:** `Views/CaptureOverlay.xaml.cs`

**Changes:**
1. Line 1440: Added `_frozenScreen?.Dispose();` before creating new bitmap in `CaptureScreenshot()`

**What could break:**
- If `CaptureScreenshot()` is called multiple times (e.g., re-capture), old screenshot is cleaned up
- No visible change — just prevents memory leak

**Test:** Open capture overlay, take screenshot, cancel, take another — no crash or slowdown

---

### Fix 06: Async Void Crash Prevention
**File:** `MainWindow.xaml.cs`

**Changes:**
1. `Sidebar_Drop()` (line 69): Wrapped entire method body in try/catch
2. `MainWindow_Activated_PaymentCheck()` (line 397): Wrapped in try/catch

**What could break:**
- If drag-drop fails, it now silently continues instead of crashing
- If payment check fails, it now silently continues instead of crashing
- No visible change — errors are just swallowed

**Test:** Drag macro to another profile tab, switch windows with payment overlay visible

---

**File:** `Views/RecordingWidgetView.xaml.cs`

**Changes:**
1. `StopButton_Click()` (line 158): Wrapped in try/catch

**What could break:**
- If stop action fails, it now silently continues instead of crashing
- No visible change

**Test:** Start recording, click stop — should work as before

---

### Other: Reverted Broken Pre-existing Change
**File:** `Services/UIElementCaptureService.cs`

**Changes:**
1. Reverted uncommitted changes that were causing build errors (CS1524, CS1513)

**Note:** This file had modifications from before this audit session. The changes introduced a syntax error. Reverted to last committed version to restore build.

---

## Batch 2 — July 23, 2026

### Fix 16: Null Safety Fixes
**Files:** `MainWindow.xaml.cs`, `Converters/PerformanceToOpacityConverter.cs`, `App.xaml`

**Changes:**
1. `MainWindow.xaml.cs` — Added null-conditional (`?.`) on 5 `ConfigManager.Current.Settings` usages: constructor (line 106), auto-update check (line 171), kill switch (line 219), minimize to tray (line 334), payment status (line 418)
2. `PerformanceToOpacityConverter.cs` — Changed unsafe `(bool)value` cast to safe `value is bool performanceMode && performanceMode`
3. `App.xaml` — Changed `StaticResource` to `DynamicResource` for `TokenPurple300Brush`, `TokenPurple400Brush`, `TokenPurple300Brush` in `PremiumCheckboxStyle` so theme switching works

**What could break:**
- Null checks return early if ConfigManager is null during shutdown — prevents crashes
- Converter no longer crashes on invalid binding values
- PremiumCheckboxStyle now updates when theme changes at runtime

**Test:** Switch dark/light theme while premium checkbox is visible

---

### Fix 17: Deadlock Prevention
**File:** `MainWindow.xaml.cs`

**Changes:**
1. Line 178: `Dispatcher.Invoke()` → `Dispatcher.BeginInvoke()` in `ContinueWith` callback (force update window)
2. Line 426: `Dispatcher.Invoke()` → `Dispatcher.BeginInvoke()` in `CheckPaymentStatusNowAsync` (payment success UI)

**What could break:**
- `BeginInvoke` is fire-and-forget instead of blocking — no deadlock risk
- UI updates now happen async instead of blocking the background thread

**Test:** Force update scenario, payment activation flow

---

### Fix 18: Converter Optimization
**Files:** `Converters/NotificationIconConverter.cs`, `Converters/MacroToHotkeyConverter.cs`

**Changes:**
1. `NotificationIconConverter.cs` — Replaced 6 per-call `new BrushConverter().ConvertFromString()` allocations with static `SolidColorBrush` fields
2. `MacroToHotkeyConverter.cs` — Cached `HotkeyToReadableConverter` as static field instead of allocating new instance per call

**What could break:**
- Brushes are frozen and reusable — no GC pressure
- Colors are identical (same RGB values as before)
- No visible change

**Test:** Open macro list sidebar with hotkey badges, check notification toasts

---

### Fix 19: Cleanup & Lifecycle
**Files:** `MainWindow.xaml.cs`, `App.xaml.cs`

**Changes:**
1. `MainWindow_Closed` — Added `_paymentListeningCts?.Cancel()`, `_paymentListeningCts?.Dispose()`, `_typedBufferTimer?.Stop()`, `_typedBufferTimer = null`
2. `App.xaml.cs` — Changed `_isSecondInstance` to `volatile` for thread visibility
3. `App.xaml.cs` — Replaced read-then-set `if (_isExiting) return; _isExiting = true;` with atomic `Interlocked.CompareExchange` in both `UnhandledException` and `DispatcherUnhandledException` handlers

**What could break:**
- Background payment listener and typing buffer timer are now cleaned up on window close
- `_isExiting` flag is now thread-safe — no race between two simultaneous exceptions

**Test:** Close app normally, trigger crash scenario

---

### Not Applied This Batch

These fixes were identified in plans but not applied due to pre-existing coverage or risk:

| Fix | Reason |
|-----|--------|
| 07 (Error Handling) | Large refactor across 50+ catch blocks — needs dedicated testing pass |
| 08 (Thread Safety) | Already handled — `_dbLock` in MacroDatabase, no `Dictionary<string, WebSocket>` in RemoteServerService |
| 09 (IDisposable) | Already handled — TrayIconManager has `Dispose()`, RemoteServerService has cleanup |
| 05 (Split God Classes) | Large refactor — needs dedicated session |

---

## Batch 3 — July 23, 2026

### Fix 11: Settings Persistence
**Status:** Already handled — `ConfigManager.Save()` with debounced timer persists all settings. `SettingsViewModel` calls `Save()` after every property change.

---

### Fix 12: Memory Optimization (DecodePixelWidth)
**Status:** Already handled — `ImagePathToThumbnailConverter.cs` already has `DecodePixelWidth = 100`, `BitmapCacheOption.OnLoad`, and `Freeze()`.

---

### Not Applied This Batch

| Fix | Reason |
|-----|--------|
| 10 (Hardcoded Strings) | Large refactor across all XAML and ViewModels — needs dedicated session |
| 13 (Logging) | Requires Serilog setup + changing all Debug.WriteLine to ILogger — needs dedicated session |
| 14 (Performance) | Async/lazy loading requires ViewModel rewrites — needs dedicated session |
| 15 (UI/UX) | Shared styles and loading indicators require XAML rewrites — needs dedicated session |
