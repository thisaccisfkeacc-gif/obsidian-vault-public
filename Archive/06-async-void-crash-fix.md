---
tags: [fix, stability, phase-1]
date: 2026-07-23
status: done
---

# 🔧 Fix 06: Async Void Crash Prevention

**Priority:** 🔴 Critical
**Effort:** 2 hours
**Risk:** 🟢 Low

---

## Problem

Multiple `async void` event handlers lack try/catch, causing **unhandled exceptions** that crash the entire app.

## Already Completed (Fix 02)

- [x] `ExecuteCaptureScreenEvent` — ✅ Fixed
- [x] `ExecuteCaptureScreenEventScope` — ✅ Fixed
- [x] `ExecuteCaptureScreenEventPixel` — ✅ Fixed
- [x] `ExecuteRunMacro` — ✅ Fixed

## Remaining (Need Fix)

### 1. HotkeyManager.cs
```csharp
// Line ~180 — Hotkey registration callback
private async void OnHotkeyPressed(int id)
{
    // Missing try/catch
    // Crashes app if hotkey handler throws
}
```

### 2. RemoteServerService.cs
```csharp
// Line ~950 — WebSocket handler
private async void HandleWebSocketConnection(WebSocket socket)
{
    // Missing try/catch
    // Crashes server if WebSocket processing fails
}

// Line ~1100 — API endpoint handler
private async void ProcessApiRequest(HttpListenerContext context)
{
    // Missing try/catch
    // Crashes server on bad request
}
```

### 3. MacroPlayer.cs
```csharp
// Line ~250 — Play macro handler
private async void PlayMacro(Macro macro)
{
    // Missing try/catch
    // Crashes app if macro execution fails
}

// Line ~320 — Record handler
private async void StartRecording()
{
    // Missing try/catch
    // Crashes app if recording fails
}
```

## Proposed Fix

Wrap all remaining async void methods:

```csharp
private async void MethodName(object sender, EventArgs e)
{
    try
    {
        // ... existing code ...
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Error in {MethodName}", nameof(MethodName));
        // Show user-friendly message if needed
    }
}
```

## Expected Impact

- **Zero unhandled async crashes**
- **App survives errors** in hotkeys, recording, remote server
- **Errors logged** instead of crashing

## Why This Is Safe

- Adding try/catch doesn't change behavior
- Just prevents crashes — normal flow unchanged
- Easy to test — trigger errors and verify no crash

---

**Awaiting Review:** Other agent to confirm "Agree" or provide counterargument.
