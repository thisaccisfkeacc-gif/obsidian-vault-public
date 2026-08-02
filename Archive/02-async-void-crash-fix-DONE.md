---
tags: [fix, stability, phase-1]
date: 2026-07-23
status: pending-review
---

# 🔧 Fix 02: async void → async Task Crash Fixes

**Priority:** 🔴 Critical
**Effort:** 1 hour
**Risk:** 🟢 Low

---

## Problem

11 `async void` methods in ViewModels can crash the entire app if an exception is thrown. In .NET, exceptions in `async void` cannot be caught by caller `try/catch` — they bubble to `SynchronizationContext` and crash the process.

## Locations Found

### File 1: `ScriptLibraryViewModel.Commands.cs`
| Line | Method |
|------|--------|
| 322 | `ExecuteCaptureAppBound` |
| 562 | `ExecuteCaptureApp` |
| 620 | `ExecuteCaptureAppFromList` |
| 661 | `ExecuteCaptureAppByCrosshair` |
| 816 | `ExecuteRunMacro` |
| 879 | `ExecuteCaptureScreenEvent` |
| 977 | `ExecuteCaptureScreenEventScope` |
| 1017 | `ExecuteCaptureScreenEventPixel` |

### File 2: `ScriptLibraryViewModel.State.cs`
| Line | Method |
|------|--------|
| 19 | `Action_PropertyChanged` |

### File 3: `TextSnippetsViewModel.cs`
| Line | Method |
|------|--------|
| 396 | `ExecuteCaptureApp` |
| 456 | `ExecuteCaptureAppFromList` |

## Proposed Fix

For each method:
1. Change `async void` → `async Task`
2. Wrap method body in `try/catch`
3. Log or show tooltip on error

**Example (before):**
```csharp
private async void ExecuteCaptureApp(object parameter)
{
    // ... async work ...
}
```

**Example (after):**
```csharp
private async Task ExecuteCaptureApp(object parameter)
{
    try
    {
        // ... async work ...
    }
    catch (Exception ex)
    {
        System.Diagnostics.Debug.WriteLine($"CaptureApp error: {ex}");
    }
}
```

**Exception:** `Action_PropertyChanged` (line 19 in State.cs) — this is an event handler, so `async void` is acceptable IF we add try/catch inside.

## Why This Is Safe

- Only changes method signatures (void → Task)
- Adds try/catch wrappers (no logic changes)
- Existing behavior preserved
- Pattern is Microsoft-recommended best practice

## Reference

- [Async/Await Best Practices (Microsoft Learn)](https://learn.microsoft.com/en-us/archive/msdn-magazine/2013/march/async-await-best-practices-in-asynchronous-programming)

---

**Awaiting Review:** Other agent to confirm "Agree" or provide counterargument.
