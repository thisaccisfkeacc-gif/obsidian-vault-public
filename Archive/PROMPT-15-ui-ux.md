---
tags: [prompt, fix-15]
date: 2026-07-23
status: ready
---

# Prompt: Fix 15 — UI/UX Consistency

## Your Task

Standardize UI patterns across the application. Write your findings and implementation to `fixes/15-ui-ux-ACTUAL.md`.

## Codebase Location

`C:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\PowerX_Keys_V2\`

## What to Do

1. **Create shared button styles** in `Styles/AppStyles.xaml`:
```xml
<Style x:Key="DangerButton" TargetType="Button">
    <Setter Property="Background" Value="{ThemeResource DangerBrush}" />
    <Setter Property="Foreground" Value="White" />
    <Setter Property="Padding" Value="12,6" />
</Style>
```

2. **Add loading indicators** to views:
```xml
<ProgressRing IsActive="{Binding IsLoading}" />
```

3. **Standardize icons** — use SymbolIcon consistently:
```xml
<Button>
    <SymbolIcon Symbol="Delete" />
</Button>
```

4. **Build and verify** — `dotnet build "c:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\PowerX_Keys_V2\PowerX_Keys_V2.csproj"`

## Key Files to Check

- All XAML files — button styles, spacing
- All ViewModels — add IsLoading property
- Resource dictionaries — shared styles

## Rules

- **Don't change functionality** — just visual consistency
- **Follow existing theme** (Dark/Light)
- **Use existing design tokens** where available
- **Build must pass** — 0 errors

## Output

Write your results to: `C:\Users\Maaz\Documents\New folder\Obsidian Vault\App Optimization & Audit\fixes\15-ui-ux-ACTUAL.md`

Include:
- Files modified
- Styles created
- Loading states added
- Build result (pass/fail)
