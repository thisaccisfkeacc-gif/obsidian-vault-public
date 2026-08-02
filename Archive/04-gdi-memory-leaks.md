---
tags: [fix, memory, phase-2]
date: 2026-07-23
status: done
---

# 🔧 Fix 04: GDI Memory Leaks

**Priority:** 🔴 Critical
**Effort:** 1.5 hours
**Risk:** 🟢 Low

---

## Problem

Windows GDI objects (pens, brushes, graphics contexts, bitmaps) are not being disposed properly, leading to:
1. Memory leaks over long sessions
2. "Generic error occurred in GDI+" exceptions
3. App crashes after extended use

## Locations Found

### 1. `ScriptEditorViewModel.cs` — Missing Dispose
```csharp
// Problem: Creates Bitmap but no Dispose
var bmp = new Bitmap(w, h);
// Should be wrapped in 'using' or explicitly disposed
```

### 2. `HotkeyManager.cs` — Icon not disposed
```csharp
// Problem: Icon creation not in using block
var icon = Icon.ExtractAssociatedIcon(path);
// Should be:
using var icon = Icon.ExtractAssociatedIcon(path);
```

### 3. `MacroStepCardViewModel.cs` — Graphics not disposed
```csharp
// Problem: Graphics object not disposed after use
var g = Graphics.FromImage(bitmap);
// Missing: g.Dispose()
```

### 4. `CaptureRegionService.cs` — Graphics leak
```csharp
// Problem: Multiple Graphics objects created, only last one disposed
var g1 = Graphics.FromImage(bitmap1);
var g2 = Graphics.FromImage(bitmap2);
// Only g2 disposed — g1 leaked
```

## Proposed Fix

Wrap all GDI objects in `using` statements:

```csharp
// Before (leaky)
var bmp = new Bitmap(w, h);
var g = Graphics.FromImage(bmp);
// ... use g ...
g.Dispose();
bmp.Dispose();

// After (safe)
using var bmp = new Bitmap(w, h);
using var g = Graphics.FromImage(bmp);
// ... use g ...
// Both auto-disposed when scope ends
```

## Files to Fix

1. `ViewModels/ScriptEditorViewModel.cs`
2. `Services/HotkeyManager.cs`
3. `ViewModels/MacroStepCardViewModel.cs`
4. `Services/CaptureRegionService.cs`
5. `Services/ThumbnailService.cs`
6. `Controls/MacroStepCardControl.xaml.cs`

## Expected Impact

- **Zero GDI leaks** after long sessions
- **No more GDI+ exceptions**
- **Stable memory usage** over time

## Why This Is Safe

- `using` statements are standard C# pattern
- No behavior change — just ensures cleanup
- Easy to verify — monitor GDI object count in Task Manager

---

**Awaiting Review:** Other agent to confirm "Agree" or provide counterargument.
