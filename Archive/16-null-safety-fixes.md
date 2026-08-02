---
tags: [fix, stability, phase-1]
date: 2026-07-23
status: done
---

# 🔧 Fix 16: Null Safety Fixes

**Priority:** 🔴 High
**Effort:** 1 hour
**Risk:** 🟢 Low

---

## Problem

Multiple null reference exception risks found during re-scan.

## Issues Found

### Issue 1: Missing null-conditional on ConfigManager (5 locations)
**File:** `MainWindow.xaml.cs` lines 104, 166, 214, 329, 409

```csharp
// Problem: ConfigManager.Current could be null during shutdown race
var settings = Services.ConfigManager.Current.Settings;

// Should be:
var settings = Services.ConfigManager.Current?.Settings;
if (settings == null) return;
```

### Issue 2: Unsafe unboxing in converter
**File:** `Converters/PerformanceToOpacityConverter.cs` line 15

```csharp
// Problem: Direct cast without type check
bool performanceMode = (bool)value;  // Throws InvalidCastException

// Should be:
if (value is bool performanceMode && performanceMode) return 0.0;
```

### Issue 3: StaticResource vs DynamicResource for themes
**File:** `App.xaml` lines 1338, 1347, 1351

```xml
<!-- Problem: StaticResource doesn't update when theme changes -->
<Setter Property="Stroke" Value="{StaticResource TokenPurple300Brush}" />

<!-- Should be: -->
<Setter Property="Stroke" Value="{DynamicResource TokenPurple300Brush}" />
```

## Files to Fix

1. `MainWindow.xaml.cs` — 5 null checks
2. `Converters/PerformanceToOpacityConverter.cs` — type check
3. `App.xaml` — 3 StaticResource → DynamicResource

## Expected Impact

- **Zero null crashes** during shutdown
- **No InvalidCastException** in converters
- **Theme switching works** for premium checkbox

---

**Awaiting Review:** Other agent to confirm "Agree" or provide counterargument.
