---
tags: [fix, lifecycle, phase-2]
date: 2026-07-23
status: done
---

# 🔧 Fix 19: Cleanup & Lifecycle Fixes

**Priority:** 🟡 Medium
**Effort:** 1 hour
**Risk:** 🟢 Low

---

## Problem

Multiple resources not properly cleaned up on shutdown, causing potential race conditions.

## Issues Found

### Issue 1: CancellationTokenSources not disposed
**File:** `MainWindow.xaml.cs` lines 345-354

```csharp
// Problem: _paymentListeningCts and _previewCts never cancelled/disposed
private void MainWindow_Closed(object sender, EventArgs e)
{
    // Missing:
    _paymentListeningCts?.Cancel();
    _paymentListeningCts?.Dispose();
    _previewCts?.Cancel();
    _previewCts?.Dispose();
}
```

### Issue 2: _typedBufferTimer not cleaned up
**File:** `MainWindow.xaml.cs` lines 489-542

```csharp
// Problem: Timer not stopped on close
// Should add to MainWindow_Closed:
_typedBufferTimer?.Stop();
_typedBufferTimer = null;
```

### Issue 3: _isExiting race condition
**File:** `App.xaml.cs` lines 31-32, 57-58

```csharp
// Problem: Read-then-set not atomic
if (_isExiting) return;
_isExiting = true;

// Should be:
if (Interlocked.CompareExchange(ref _isExiting, true, false)) return;
```

### Issue 4: _isSecondInstance missing volatile
**File:** `App.xaml.cs` line 21

```csharp
// Problem: Visibility across threads not guaranteed
internal static bool _isSecondInstance = false;

// Should be:
internal static volatile bool _isSecondInstance = false;
```

## Files to Fix

1. `MainWindow.xaml.cs` — CTS cleanup + timer cleanup
2. `App.xaml.cs` — Interlocked for _isExiting + volatile for _isSecondInstance

## Expected Impact

- **Clean shutdown** — no orphaned tasks
- **No race conditions** — atomic flag operations
- **Predictable cleanup** — all resources released

---

**Awaiting Review:** Other agent to confirm "Agree" or provide counterargument.
