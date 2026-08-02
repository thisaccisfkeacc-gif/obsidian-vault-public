---
tags: [prompt, fix-13]
date: 2026-07-23
status: ready
---

# Prompt: Fix 13 — Standardize Logging

## Your Task

Standardize logging across the codebase using Serilog. Write your findings and implementation to `fixes/13-logging-ACTUAL.md`.

## Codebase Location

`C:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\PowerX_Keys_V2\`

## What to Do

1. **Configure Serilog** in `App.xaml.cs`:
```csharp
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .WriteTo.File("logs/powerx-keys-.log", 
        rollingInterval: RollingInterval.Day,
        retainedFileCountLimit: 7)
    .CreateLogger();
```

2. **Add ILogger to services** that use `Debug.WriteLine`:
```csharp
// Before
Debug.WriteLine("Macro saved: " + macro.Name);

// After
_logger.LogInformation("Macro saved: {MacroName}", macro.Name);
```

3. **Replace Console.WriteLine** (useless in WPF):
```csharp
// Before
Console.WriteLine("Starting server...");

// After  
_logger.LogInformation("Starting server on port {Port}", port);
```

4. **Build and verify** — `dotnet build "c:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\PowerX_Keys_V2\PowerX_Keys_V2.csproj"`

## Key Files to Check

- `App.xaml.cs` — Serilog configuration
- `Services/RemoteServerService.cs` — Console.WriteLine calls
- `Managers/MacroDatabase.cs` — Debug.WriteLine calls
- All services with `Debug.WriteLine`

## Rules

- **Use ILogger injection** where possible
- **Use Serilog structured logging** (named parameters, not string interpolation)
- **Don't change behavior** — just improve logging
- **Build must pass** — 0 errors

## Output

Write your results to: `C:\Users\Maaz\Documents\New folder\Obsidian Vault\App Optimization & Audit\fixes\13-logging-ACTUAL.md`

Include:
- Files modified
- Debug.WriteLine replaced (count)
- Console.WriteLine replaced (count)
- Build result (pass/fail)
