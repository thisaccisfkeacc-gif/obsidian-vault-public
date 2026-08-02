---
tags: [fix, memory, phase-2]
date: 2026-07-23
status: done
---

# 🔧 Fix 09: Implement IDisposable Pattern

**Priority:** 🟡 Medium
**Effort:** 2 hours
**Risk:** 🟢 Low

---

## Problem

Multiple services create resources (timers, connections, watchers) but don't implement `IDisposable`, causing memory leaks when the app shuts down.

## Locations Found

### 1. HotkeyManager.cs
```csharp
// Problem: Global hotkey hooks not cleaned up
private IntPtr _hookId;
// No Dispose method — hooks leak
```

### 2. MacroPlayer.cs
```csharp
// Problem: Timers not disposed
private DispatcherTimer _playbackTimer;
// No cleanup on shutdown
```

### 3. RemoteServerService.cs
```csharp
// Problem: HttpListener not disposed
private HttpListener _listener;
// WebSocket connections not closed properly
```

### 4. FileWatcherService.cs
```csharp
// Problem: FileSystemWatcher not disposed
private FileSystemWatcher _watcher;
// Continues watching after app supposed to stop
```

## Proposed Fix

Implement `IDisposable` on all services:

```csharp
public class HotkeyManager : IDisposable
{
    private bool _disposed;
    
    public void Dispose()
    {
        if (!_disposed)
        {
            UnhookWindowsHookEx(_hookId);
            _disposed = true;
        }
    }
}
```

Register for app shutdown cleanup:
```csharp
// In App.xaml.cs
protected override void OnExit(ExitEventArgs e)
{
    _hotkeyManager?.Dispose();
    _macroPlayer?.Dispose();
    _remoteServer?.Dispose();
    base.OnExit(e);
}
```

## Files to Update

1. `Services/HotkeyManager.cs`
2. `Services/MacroPlayer.cs`
3. `Services/RemoteServerService.cs`
4. `Services/FileWatcherService.cs`
5. `App.xaml.cs` — add cleanup calls

## Expected Impact

- **Clean shutdown** — no resource leaks
- **Proper hook removal** — no orphaned global hooks
- **Stable system** — no zombie processes

---

**Awaiting Review:** Other agent to confirm "Agree" or provide counterargument.
