---
tags: [prompt, fix-07]
date: 2026-07-23
status: ready
---

# Prompt: Fix 07 — Standardize Error Handling

## Your Task

Standardize error handling across the codebase. Write your findings and implementation to `fixes/07-error-handling-ACTUAL.md`.

## Codebase Location

`C:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\PowerX_Keys_V2\`

## What to Do

1. **Create `Services/ErrorHelper.cs`** utility class:
```csharp
public static class ErrorHelper
{
    public static void HandleError(Exception ex, string context, ILogger logger)
    {
        logger.LogError(ex, "Error in {Context}", context);
    }
}
```

2. **Find all empty catch blocks** and add logging:
```csharp
// Before
catch (Exception) { }

// After
catch (Exception ex)
{
    ErrorHelper.HandleError(ex, nameof(MethodName), _logger);
}
```

3. **Find all raw Exception throws** and add context:
```csharp
// Before
throw new Exception("Failed to save macro");

// After
throw new Exception($"Failed to save macro in {nameof(SaveMacro)}", ex);
```

4. **Build and verify** — `dotnet build "c:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\PowerX_Keys_V2\PowerX_Keys_V2.csproj"`

## Key Files to Check

- `Services/RemoteServerService.cs` — 15+ empty catches
- `Managers/MacroDatabase.cs` — 8+ inconsistent catches
- `ViewModels/ScriptLibraryViewModel.cs` — 12+ raw throws
- `Services/HotkeyManager.cs` — 5+ silent failures

## Rules

- **Don't change behavior** — just add logging
- **Use ILogger** where available, fall back to DebugLogger
- **Don't break existing catch blocks** that have valid logic
- **Build must pass** — 0 errors

## Output

Write your results to: `C:\Users\Maaz\Documents\New folder\Obsidian Vault\App Optimization & Audit\fixes\07-error-handling-ACTUAL.md`

Include:
- Files modified
- Empty catches fixed (count)
- Raw throws fixed (count)
- Build result (pass/fail)
