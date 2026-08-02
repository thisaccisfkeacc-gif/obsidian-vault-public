---
tags: [prompt, fix-10]
date: 2026-07-23
status: ready
---

# Prompt: Fix 10 — Extract Hardcoded Strings

## Your Task

Extract hardcoded UI strings to resource files for future localization. Write your findings and implementation to `fixes/10-hardcoded-strings-ACTUAL.md`.

## Codebase Location

`C:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\PowerX_Keys_V2\`

## What to Do

1. **Create `Properties/Resources.resx`** with common strings:
```xml
<data name="Button_Delete" xml:space="preserve">
    <value>Delete</value>
</data>
<data name="Button_Edit" xml:space="preserve">
    <value>Edit</value>
</data>
<data name="Error_FailedToLoadMacro" xml:space="preserve">
    <value>Failed to load macro. Please try again.</value>
</data>
```

2. **Update XAML files**:
```xml
<!-- Before -->
<Button Content="Delete" />

<!-- After -->
<Button Content="{x:Static res:Resources.Button_Delete}" />
```

3. **Update C# files**:
```csharp
// Before
throw new Exception("Failed to load macro");

// After
throw new Exception(Resources.Error_FailedToLoadMacro);
```

4. **Build and verify** — `dotnet build "c:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\PowerX_Keys_V2\PowerX_Keys_V2.csproj"`

## Key Files to Check

- All XAML files — button labels, headers
- All ViewModels — error messages, dialog text
- All Services — status messages

## Rules

- **Only extract user-visible strings** (not log messages)
- **Don't change behavior** — just move text to resources
- **Use descriptive resource names** (Button_Delete, Error_X, etc.)
- **Build must pass** — 0 errors

## Output

Write your results to: `C:\Users\Maaz\Documents\New folder\Obsidian Vault\App Optimization & Audit\fixes\10-hardcoded-strings-ACTUAL.md`

Include:
- Files modified
- Strings extracted (count)
- Resource file created/updated
- Build result (pass/fail)
