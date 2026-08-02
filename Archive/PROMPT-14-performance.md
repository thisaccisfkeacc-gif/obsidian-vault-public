---
tags: [prompt, fix-14]
date: 2026-07-23
status: ready
---

# Prompt: Fix 14 — Performance Optimization

## Your Task

Improve app performance with async loading and lazy initialization. Write your findings and implementation to `fixes/14-performance-ACTUAL.md`.

## Codebase Location

`C:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\PowerX_Keys_V2\`

## What to Do

1. **Make database calls async** where they block UI:
```csharp
// Before
var macros = GetAllMacros(); // Blocks UI

// After
var macros = await Task.Run(() => GetAllMacros());
```

2. **Add loading states** to ViewModels:
```csharp
private bool _isLoading;
public bool IsLoading
{
    get => _isLoading;
    set { _isLoading = value; OnPropertyChanged(); }
}
```

3. **Lazy-load heavy resources** on first access instead of constructor

4. **Build and verify** — `dotnet build "c:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\PowerX_Keys_V2\PowerX_Keys_V2.csproj"`

## Key Files to Check

- `ViewModels/ScriptLibraryViewModel.cs` — Loads all macros on startup
- `Managers/MacroDatabase.cs` — Synchronous database calls
- `ViewModels/MainViewModel.cs` — Heavy initialization

## Rules

- **Don't change public API** unless necessary
- **Keep UI responsive** — move heavy work to background
- **Add cancellation support** where practical
- **Build must pass** — 0 errors

## Output

Write your results to: `C:\Users\Maaz\Documents\New folder\Obsidian Vault\App Optimization & Audit\fixes\14-performance-ACTUAL.md`

Include:
- Files modified
- Async conversions made
- Loading states added
- Build result (pass/fail)
