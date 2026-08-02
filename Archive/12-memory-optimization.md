---
tags: [performance, memory, phase-3]
date: 2026-07-23
status: done
---

# 🔧 Fix 12: Memory Optimization

**Priority:** 🟡 Medium
**Effort:** 2 hours
**Risk:** 🟢 Low

---

## Problem

Large objects kept in memory unnecessarily, causing:
1. High memory usage over time
2. Sluggish UI when many macros loaded
3. Thumbnail images never released

## Locations Found

### 1. MacroDatabase.cs — All Macros Loaded
```csharp
// Problem: Loads ALL macros into memory
public List<Macro> GetAllMacros()
{
    // Returns 1000+ macros at once
    // Should lazy-load or paginate
}
```

### 2. ThumbnailService.cs — Thumbnails Cached Forever
```csharp
// Problem: Thumbnails cached, never released
private Dictionary<string, Bitmap> _thumbnailCache;
// Thumbnails accumulate, never cleaned
```

### 3. ScriptViewModel.cs — Event List Grows
```csharp
// Problem: Recorded events kept forever
private List<EventItem> _recordedEvents;
// Recording 10 minutes = thousands of events in memory
```

## Proposed Fix

### Part A: Lazy-load macros
```csharp
public async Task<List<Macro>> GetMacrosAsync(int page = 0, int pageSize = 50)
{
    return await _connection.QueryAsync<Macro>(
        "SELECT * FROM Macros LIMIT @PageSize OFFSET @Offset",
        new { PageSize = pageSize, Offset = page * pageSize });
}
```

### Part B: Limit thumbnail cache
```csharp
private const int MaxCacheSize = 100;

public void CacheThumbnail(string key, Bitmap thumbnail)
{
    if (_thumbnailCache.Count >= MaxCacheSize)
    {
        // Remove oldest entry
        var oldest = _thumbnailCache.Keys.First();
        _thumbnailCache[oldest].Dispose();
        _thumbnailCache.Remove(oldest);
    }
    _thumbnailCache[key] = thumbnail;
}
```

### Part C: Clear events after recording
```csharp
public void StopRecording()
{
    // Process events, then clear list
    ProcessRecordedEvents(_recordedEvents);
    _recordedEvents.Clear(); // Free memory
}
```

### Part D: WPF `DecodePixelWidth` for Thumbnails
When loading image step thumbnails into WPF controls, set `DecodePixelWidth = 200` to decode images at UI size rather than storing full 4K resolutions in RAM.

## Expected Impact

- **50% less memory** during normal use
- **Faster UI** — less data to render
- **Stable long sessions** — no memory growth

---

**Awaiting Review:** Other agent to confirm "Agree" or provide counterargument.
