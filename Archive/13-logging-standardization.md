---
tags: [refactor, debugging, phase-3]
date: 2026-07-23
status: done
---

# 🔧 Fix 13: Standardize Logging

**Priority:** 🟡 Medium
**Effort:** 1.5 hours
**Risk:** 🟢 Low

---

## Problem

Logging is inconsistent across the codebase:
1. Some places use `_logger` (ILogger)
2. Others use `Debug.WriteLine`
3. Others use `Console.WriteLine`
4. Some have no logging at all

## Current State

```csharp
// Pattern 1: ILogger (good)
_logger.LogError(ex, "Failed to save");

// Pattern 2: Debug (only in debug mode)
Debug.WriteLine("Error: " + ex.Message);

// Pattern 3: Console (useless in WPF)
Console.WriteLine("Starting server...");

// Pattern 4: Nothing
catch (Exception) { } // Silent failure
```

## Proposed Fix

### Part A: Configure Serilog
```csharp
// In App.xaml.cs
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .WriteTo.File("logs/powerx-keys-.log", 
        rollingInterval: RollingInterval.Day,
        retainedFileCountLimit: 7)
    .CreateLogger();
```

### Part B: Inject ILogger everywhere
```csharp
public class MacroDatabase
{
    private readonly ILogger<MacroDatabase> _logger;
    
    public MacroDatabase(ILogger<MacroDatabase> logger)
    {
        _logger = logger;
    }
}
```

### Part C: Replace all Debug.WriteLine
```csharp
// Before
Debug.WriteLine("Macro saved: " + macro.Name);

// After
_logger.LogInformation("Macro saved: {MacroName}", macro.Name);
```

### Part D: Remove empty catches
```csharp
// Before
catch (Exception) { }

// After
catch (Exception ex)
{
    _logger.LogWarning(ex, "Error in {Method}", nameof(MethodName));
}
```

## Expected Impact

- **Centralized logs** — all in one file
- **Better debugging** — can filter by level
- **Production insights** — logs available when issues occur

---

**Awaiting Review:** Other agent to confirm "Agree" or provide counterargument.
