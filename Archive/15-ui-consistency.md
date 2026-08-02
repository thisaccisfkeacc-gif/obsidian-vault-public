---
tags: [ux, polish, phase-4]
date: 2026-07-23
status: done
---

# 🔧 Fix 15: UI/UX Consistency

**Priority:** 🟢 Low
**Effort:** 2 hours
**Risk:** 🟢 Low

---

## Problem

Inconsistent UI patterns across the application:
1. Different button styles in different views
2. Inconsistent spacing and margins
3. Mixed icon styles
4. No loading indicators

## Current Issues

### 1. Button Styles
```xml
<!-- Different styles everywhere -->
<Button Background="Red" Foreground="White" /> <!-- One style -->
<Button Style="{StaticResource PrimaryButton}" /> <!-- Another style -->
<Button BorderBrush="Gray" /> <!-- Yet another -->
```

### 2. No Loading States
```csharp
// Problem: UI freezes while loading
public void LoadMacros()
{
    // No loading indicator
    // User sees frozen screen
}
```

### 3. Inconsistent Icons
```xml
<!-- Mix of icon sources -->
<Image Source="icon_delete.png" />
<SymbolIcon Symbol="Delete" />
<Path Data="M..." />
```

## Proposed Fix

### Part A: Create shared styles
```xml
<!-- Styles/AppStyles.xaml -->
<Style x:Key="DangerButton" TargetType="Button">
    <Setter Property="Background" Value="{ThemeResource DangerBrush}" />
    <Setter Property="Foreground" Value="White" />
    <Setter Property="Padding" Value="12,6" />
</Style>
```

### Part B: Add loading indicators
```xml
<!-- Show spinner during loading -->
<ProgressRing IsActive="{Binding IsLoading}" />
<ItemsControl Visibility="{Binding IsLoading, Converter={StaticResource InvertBool}}">
    <!-- Content here -->
</ItemsControl>
```

### Part C: Standardize icons
```xml
<!-- Use SymbolIcon consistently -->
<Button>
    <SymbolIcon Symbol="Delete" />
</Button>
```

## Files to Update

1. All XAML files — button styles
2. All ViewModels — add IsLoading property
3. Resource dictionaries — shared styles

## Expected Impact

- **Professional look** — consistent UI
- **Better feedback** — loading states
- **Easier maintenance** — shared styles

---

**Awaiting Review:** Other agent to confirm "Agree" or provide counterargument.
