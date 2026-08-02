---
tags: [fix, performance, phase-2]
date: 2026-07-23
status: done
---

# 🔧 Fix 03: SQLite WAL Mode + Transactions

**Priority:** 🟡 High
**Effort:** 1 hour
**Risk:** 🟢 Low

---

## Problem

SQLite operations are slow because:
1. No WAL mode (Write-Ahead Logging) — default is DELETE journal mode
2. No explicit transactions for batch operations — each INSERT/UPDATE is its own transaction
3. No busy_timeout — causes "Database is locked" errors under concurrent access

## Location

`Managers/MacroDatabase.cs` — connection setup and CRUD operations

## Current Issues

### Issue 1: No WAL Mode
```csharp
// Current: No journal mode set
// Should be:
PRAGMA journal_mode=WAL;
PRAGMA synchronous=NORMAL;
PRAGMA busy_timeout=5000;
```

### Issue 2: No Batch Transactions
Methods like `SaveMacro`, `SaveStep`, `LoadAllMacros` do individual INSERT/UPDATE without wrapping in a transaction.

## Proposed Fix

### Part A: Add PRAGMAs on connection open
After opening SQLite connection, execute:
```csharp
using var cmd = connection.CreateCommand();
cmd.CommandText = @"
    PRAGMA journal_mode=WAL;
    PRAGMA synchronous=NORMAL;
    PRAGMA busy_timeout=5000;
    PRAGMA temp_store=MEMORY;";
cmd.ExecuteNonQuery();
```

### Part B: Wrap batch operations in transactions
For methods that do multiple INSERTs/UPDATEs:
```csharp
using var transaction = connection.BeginTransaction();
try
{
    // ... multiple INSERT/UPDATE statements ...
    transaction.Commit();
}
catch
{
    transaction.Rollback();
    throw;
}
```

### Methods to wrap:
1. `SaveMacro` — saves macro + all steps
2. `SaveStep` — saves step + children
3. `LoadAllMacros` — reads all macros (read transaction)
4. `DeleteMacro` — deletes macro + children
5. `ReorderSteps` — updates multiple step positions

## Expected Impact

- **10-50x faster** macro saves
- **No more "Database is locked"** errors
- **Smoother recording** — steps save instantly

## Why This Is Safe

- PRAGMAs are standard SQLite best practices
- WAL mode is backwards compatible
- Transactions are already used in some places (just not consistently)
- Easy to test — save/load macros before and after

## Reference

- [Microsoft.Data.Sqlite Performance Best Practices](https://learn.microsoft.com/en-us/dotnet/standard/data/sqlite/async)

---

**Awaiting Review:** Other agent to confirm "Agree" or provide counterargument.
