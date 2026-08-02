---
tags: [fix, concurrency, phase-3]
date: 2026-07-23
status: done
---

# 🔧 Fix 08: Thread Safety for Shared Resources

**Priority:** 🟡 Medium
**Effort:** 1.5 hours
**Risk:** 🟡 Medium (concurrency changes)

---

## Problem

Multiple threads access shared resources without proper synchronization, causing potential race conditions and data corruption.

## Locations Found

### 1. RemoteServerService.cs — Concurrent Collections
```csharp
// Problem: Dictionary modified from multiple threads
private Dictionary<string, WebSocket> _clients = new();

// Multiple threads add/remove simultaneously
// Can cause "Collection was modified during enumeration" exceptions
```

### 2. MacroDatabase.cs — Connection Sharing
```csharp
// Problem: Single SQLite connection used across threads
private SqliteConnection _connection;

// Multiple async operations share connection
// Can cause "Database is locked" errors
```

### 3. ScriptViewModel.cs — Event List
```csharp
// Problem: Event list modified during execution
private ObservableCollection<EventItem> _events;

// Recording thread adds events while execution reads them
// Can cause index out of range exceptions
```

## Proposed Fix

### Part A: Use ConcurrentDictionary for clients
```csharp
// Before
private Dictionary<string, WebSocket> _clients = new();

// After
private ConcurrentDictionary<string, WebSocket> _clients = new();
```

### Part B: Use SemaphoreSlim for database
```csharp
private readonly SemaphoreSlim _dbLock = new(1, 1);

public async Task<Macro> SaveMacroAsync(Macro macro)
{
    await _dbLock.WaitAsync();
    try
    {
        // ... database operations ...
    }
    finally
    {
        _dbLock.Release();
    }
}
```

### Part C: Use lock for event list access
```csharp
private readonly object _eventLock = new();

public void AddEvent(EventItem item)
{
    lock (_eventLock)
    {
        _events.Add(item);
    }
}
```

## Expected Impact

- **No more race conditions**
- **No more "Collection modified" exceptions**
- **Stable concurrent operations**

## Risk Mitigation

- Use well-tested synchronization primitives
- Avoid deadlocks by locking in consistent order
- Test with concurrent operations

---

**Awaiting Review:** Other agent to confirm "Agree" or provide counterargument.
