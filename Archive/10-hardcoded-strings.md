---
tags: [refactor, i18n, phase-4]
date: 2026-07-23
status: done
---

# 🔧 Fix 10: Extract Hardcoded Strings

**Priority:** 🟢 Low
**Effort:** 2 hours
**Risk:** 🟢 Low

---

## Problem

UI text, error messages, and button labels are hardcoded throughout the codebase, making localization impossible.

## Locations Found

### 1. XAML Files — Button Labels
```xml
<!-- Controls/MacroStepCardControl.xaml -->
<Button Content="Delete" />
<Button Content="Edit" />
<Button Content="Run" />
```

### 2. ViewModel Files — Error Messages
```csharp
// ScriptLibraryViewModel.cs
throw new Exception("Failed to load macro");
MessageBox.Show("Are you sure?");
```

### 3. Service Files — Status Messages
```csharp
// RemoteServerService.cs
"Server started on port {0}"
"Client connected"
"Error processing request"
```

## Proposed Fix

### Part A: Create Resources.resx
```xml
<!-- Properties/Resources.resx -->
<data name="Button_Delete" xml:space="preserve">
    <value>Delete</value>
</data>
<data name="Error_FailedToLoadMacro" xml:space="preserve">
    <value>Failed to load macro. Please try again.</value>
</data>
```

### Part B: Update XAML
```xml
<!-- Before -->
<Button Content="Delete" />

<!-- After -->
<Button Content="{x:Static res:Resources.Button_Delete}" />
```

### Part C: Update C#
```csharp
// Before
throw new Exception("Failed to load macro");

// After
throw new Exception(Resources.Error_FailedToLoadMacro);
```

## Files to Update

1. All XAML files — button labels, headers
2. All ViewModels — error messages, dialog text
3. All Services — status messages, log messages

## Expected Impact

- **Localization ready** — can add languages later
- **Consistent text** — no typos from duplicates
- **Easier maintenance** — text in one place

## Why This Is Safe

- Purely string extraction
- No behavior changes
- Can be done incrementally

---

**Awaiting Review:** Other agent to confirm "Agree" or provide counterargument.
