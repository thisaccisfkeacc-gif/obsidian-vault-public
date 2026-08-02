---
tags: [performance, memory, phase-3]
date: 2026-07-23
status: done
---

# 🔧 Fix 18: Converter Optimization

**Priority:** 🟡 Medium
**Effort:** 1 hour
**Risk:** 🟢 Low

---

## Problem

Value converters allocate new objects on every call, causing GC pressure.

## Issues Found

### Issue 1: BrushConverter allocated per call
**File:** `Converters/NotificationIconConverter.cs` lines 35-40

```csharp
// Problem: New BrushConverter created every time
"Warning" => (Brush)new BrushConverter().ConvertFromString("#CCAA00"),
```

### Issue 2: HotkeyToReadableConverter allocated per call
**File:** `Converters/MacroToHotkeyConverter.cs` line 40

```csharp
// Problem: New converter created every binding evaluation
var converter = new HotkeyToReadableConverter();
```

### Issue 3: ImagePathToThumbnailConverter reads entire file
**File:** `Converters/ImagePathToThumbnailConverter.cs` line 31

```csharp
// Problem: Reads entire file into memory
byte[] bytes = File.ReadAllBytes(path);
// Should stream directly
```

## Proposed Fix

### Part A: Cache brushes as static fields
```csharp
private static readonly Brush WarningBrush = new SolidColorBrush(Color.FromRgb(0xCC, 0xAA, 0x00));
private static readonly Brush ErrorBrush = new SolidColorBrush(Color.FromRgb(0xCC, 0x33, 0x33));
// ... etc
```

### Part B: Reuse converter instances
```csharp
private static readonly HotkeyToReadableConverter _hotkeyConverter = new();

// In Convert():
return _hotkeyConverter.Convert(raw, typeof(string), null, CultureInfo.CurrentCulture);
```

### Part C: Stream files directly
```csharp
// Before
byte[] bytes = File.ReadAllBytes(path);

// After
using var stream = File.OpenRead(path);
var bitmap = new BitmapImage();
bitmap.BeginInit();
bitmap.StreamSource = stream;
bitmap.CacheOption = BitmapCacheOption.OnLoad;
bitmap.EndInit();
bitmap.Freeze();
```

## Files to Fix

1. `Converters/NotificationIconConverter.cs`
2. `Converters/MacroToHotkeyConverter.cs`
3. `Converters/ImagePathToThumbnailConverter.cs`

## Expected Impact

- **Less GC pressure** — no per-call allocations
- **Faster rendering** — cached objects reused
- **Lower memory** — no duplicate byte arrays

---

**Awaiting Review:** Other agent to confirm "Agree" or provide counterargument.
