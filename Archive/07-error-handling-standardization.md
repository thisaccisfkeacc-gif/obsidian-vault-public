---
tags: [refactor, reliability, phase-3]
date: 2026-07-23
status: done
---

# 🔧 Fix 07: Standardize Error Handling

**Priority:** 🟡 Medium
**Effort:** 2 hours
**Risk:** 🟢 Low

---

## Problem

Error handling is inconsistent across the codebase:
1. Some methods swallow errors silently
2. Some throw raw exceptions
3. Some use custom error types
4. No centralized error reporting

## Current Issues

### Issue 1: Silent Failures
```csharp
// RemoteServerService.cs
catch (Exception)
{
    // Silently ignored — user never knows what happened
}
```

### Issue 2: Raw Exception Throws
```csharp
// MacroDatabase.cs
throw new Exception("Failed to save macro");
// No inner exception, no context
```

### Issue 3: Inconsistent Logging
```csharp
// Some places use _logger
_logger.LogError(ex, "Failed");

// Others use Debug.WriteLine
Debug.WriteLine("Error: " + ex.Message);

// Others have no logging at all
```

## Proposed Fix

### Part A: Create ErrorHelper utility
```csharp
// Services/ErrorHelper.cs
public static class ErrorHelper
{
    public static void HandleError(Exception ex, string context, ILogger logger)
    {
        logger.LogError(ex, "Error in {Context}", context);
        // Optionally show user notification
    }
    
    public static T HandleError<T>(Exception ex, string context, ILogger logger, T defaultValue)
    {
        HandleError(ex, context, logger);
        return defaultValue;
    }
}
```

### Part B: Standardize all catch blocks
```csharp
// Before
catch (Exception ex)
{
    Debug.WriteLine("Error: " + ex.Message);
}

// After
catch (Exception ex)
{
    ErrorHelper.HandleError(ex, nameof(MethodName), _logger);
}
```

### Part C: Remove silent catches
Replace empty catch blocks with logged catches.

## Files to Update

1. `Services/RemoteServerService.cs` — 15+ empty catches
2. `Managers/MacroDatabase.cs` — 8+ inconsistent catches
3. `ViewModels/ScriptLibraryViewModel.cs` — 12+ raw throws
4. `Services/HotkeyManager.cs` — 5+ silent failures

## Expected Impact

- **All errors logged** — easier debugging
- **No silent failures** — user sees what went wrong
- **Consistent pattern** — easier to maintain

## Why This Is Safe

- Just changing how errors are reported
- No behavior changes
- Easy to verify — grep for empty catch blocks

---

**Awaiting Review:** Other agent to confirm "Agree" or provide counterargument.
