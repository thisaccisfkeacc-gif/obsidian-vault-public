---
tags: [fix, stability, phase-1]
date: 2026-07-23
status: done
---

# 🔧 Fix 17: Deadlock Prevention

**Priority:** 🔴 High
**Effort:** 30 minutes
**Risk:** 🟢 Low

---

## Problem

Synchronous `Dispatcher.Invoke()` calls from background threads can deadlock the UI.

## Location

**File:** `MainWindow.xaml.cs` lines 173 and 414

```csharp
// Problem: Dispatcher.Invoke() blocks the calling thread
// If UI thread is blocked (e.g., modal dialog), deadlock occurs
Dispatcher.Invoke(() => { ... });
```

## Proposed Fix

Replace `Dispatcher.Invoke()` with `Dispatcher.BeginInvoke()`:

```csharp
// Before (deadlock risk)
Dispatcher.Invoke(() => { ... });

// After (safe, fire-and-forget)
Dispatcher.BeginInvoke(() => { ... });
```

Or use the modern pattern:
```csharp
Application.Current?.Dispatcher?.BeginInvoke(() => { ... });
```

## Files to Fix

1. `MainWindow.xaml.cs` — 2 locations

## Expected Impact

- **No more deadlocks** from background-to-UI calls
- **Smoother operation** during concurrent operations

---

**Awaiting Review:** Other agent to confirm "Agree" or provide counterargument.
