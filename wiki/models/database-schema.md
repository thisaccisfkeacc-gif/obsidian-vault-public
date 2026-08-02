---
tags: [model, database, sqlite, schema]
date: 2026-08-01
sources:
  - Managers/MacroDatabase.cs
status: current
---

# Database Schema 🗄️

PowerX Keys uses **SQLite** for macro storage. The database is managed by `MacroDatabase`, a static class with direct SQL operations.

## Database Location

```
%LOCALAPPDATA%/PowerXKeys/Configs/macros.db
```

(Note the `Configs` subfolder — `MacroDatabase.cs:15-16`.)

## Tables

### `Macros`

The primary table storing all user macros. Base columns `Id`, `Name`, `Icon`; the rest are added non-destructively via `EnsureColumns()` migrations.

| Column | Type | Purpose |
|--------|------|---------|
| `Id` | TEXT (PK) | Guid string |
| `Name` | TEXT | Macro display name |
| `Icon` | TEXT | Emoji icon |
| `IsFavorite` | INTEGER | Boolean |
| `PlaybackSpeed` | REAL | Speed multiplier (default 1.0) |
| `MousePhysicsProfile` | INTEGER | Physics profile ID |
| `TraceCaptureMode` | INTEGER | Trace mode |
| `BlockHardwareInput` | INTEGER | Boolean |
| `AutoDelayEnabled` | INTEGER | Boolean |
| `AutoDelayMs` | INTEGER | Delay in ms (default 100) |
| `IsHumanized` | INTEGER | Boolean (always saved as off — non-persistent) |
| `DefaultHumanizationLevel` | INTEGER | Humanization level |

There is **no `AssignedProfile` column** — profiles are managed in `config.json`, not the database.

### `MacroSteps`

Stores individual macro steps as distinct rows, linked to their parent macro. Extended properties are stored as JSON.

| Column | Type | Purpose |
|--------|------|---------|
| `Id` | TEXT (PK) | Step Guid |
| `MacroId` | TEXT | FK to Macros(Id) (ON DELETE CASCADE) |
| `Type` | INTEGER | MacroStepType enum |
| `Value` | TEXT | Core value (e.g. text or key combo) |
| `X` | REAL | Coordinate X |
| `Y` | REAL | Coordinate Y |
| `Duration` | INTEGER | Delay or timeout in ms |
| `TraceFileId` | TEXT | Guid for mouse trace |
| `SearchImageFilename` | TEXT | Image file reference |
| `SearchScopeSummary` | TEXT | Search region bounds |
| `IsDebugHighlight` | INTEGER | Boolean |
| `ParentId` | TEXT | Used for nested blocks (If/Group) |
| `OrderIndex` | INTEGER | Execution order index |
| `IsFalseBranch` | INTEGER | 1 if under Else branch, else 0 |
| `TargetColorHex` | TEXT | Pixel color |
| `Precision` | INTEGER | Deprecated (always 3) |
| `Tolerance` | INTEGER | Pixel Tolerance |
| `SearchWidth` | REAL | Search box width |
| `SearchHeight` | REAL | Search box height |
| `WindowTitle` | TEXT | Captured window title |
| `KeyActionType` | TEXT | "Press", "Hold Down", "Released Up" |
| `ClickCount` | INTEGER | Click count / loop iterations |
| `TimerInterval` | INTEGER | Timer click interval (ms, default 100) |
| `DoubleClickSpeed` | INTEGER | Double-click speed (ms, default 300) |
| `IsManuallyAdded` | INTEGER | Boolean |
| `ExtraJson` | TEXT | JSON blob for all remaining properties |

### `CaptureLibrary_UIElements`
Stores captured UI Automation elements. See details in [[capture-library]].
Columns: `Id`, `ElementName`, `AutomationId`, `ClassName`, `ControlType`, `WindowTitle`, `ElementPath`, `ScreenshotPath`, `X`, `Y`, `CapturedAt`, `LastUsedAt`, `UseCount`, `IsFavorite` — plus `ProcessName` (added by migration in `MacroDatabase.cs:142-149`).

### `CaptureLibrary_Images`
Stores captured visual search images. See details in [[capture-library]].
Columns: `Id`, `ImagePath`, `Label`, `SourceApp`, `Width`, `Height`, `CapturedAt`, `LastUsedAt`, `UseCount`, `IsFavorite`.

### `CaptureLibrary_Pixels`
Stores captured pixel color targets. See details in [[capture-library]].
Columns: `Id`, `ColorHex`, `Label`, `SourceApp`, `X`, `Y`, `CapturedAt`, `LastUsedAt`, `UseCount`, `IsFavorite`.

### `CaptureLibrary_Windows`
Stores captured window targets for Window Action steps (created in `MacroDatabase.cs:185-199`).

| Column | Type | Purpose |
|--------|------|---------|
| `Id` | TEXT (PK) | Guid |
| `AppName` | TEXT | App display name |
| `WindowTitle` | TEXT | Window title |
| `ProcessName` | TEXT | Process exe name |
| `MatchMode` | TEXT | Default 'AnyWindow' |
| `CapturedAt` / `LastUsedAt` | TEXT | Timestamps |
| `UseCount` | INTEGER | Usage counter |
| `IsFavorite` | INTEGER | Boolean |

## Key Design Decisions

### Step Serialization strategy

Instead of a single JSON blob for the entire macro, each step is an individual row in `MacroSteps`. Core fields are normal columns for performance, while 100+ extended fields (UI parameters, logic branches, etc.) are serialized into the `ExtraJson` column. This keeps the table schema maintainable while preserving complex state.

### No ORM

`MacroDatabase` uses raw `Microsoft.Data.Sqlite` ADO.NET calls — no Entity Framework or Dapper. This keeps the dependency footprint minimal.

## CRUD Operations

### `SaveMacro(MacroItem macro)`
- Begins an explicit transaction.
- **UPDATE-or-INSERT**: checks whether the macro exists (`SELECT Name FROM Macros WHERE Id = $id`); if found → `UPDATE Macros SET ...`, otherwise → `INSERT INTO Macros ...`.
- Deletes all existing `MacroSteps` rows for the macro, then re-inserts via a recursive tree walk (`SaveStep` handles `ChildSteps` and `ChildStepsFalse`).
- Serializes secondary properties into `ExtraJson` via `System.Text.Json`.

### `LoadAllMacros() → ObservableCollection<MacroItem>`
- `SELECT ... FROM Macros`
- Hydrates steps from `MacroSteps` (SELECT ordered by `OrderIndex ASC`) and parses the `ExtraJson` property dictionary.
- Reconstructs the tree structure for nested steps using `ParentId` + `IsFalseBranch`.
- Caches the result (invalidated by `ClearCache()` on every save/delete).

### `DeleteMacro(Guid id)`
- `DELETE FROM Macros WHERE Id = @id` (Cascade deletes `MacroSteps` via FK).
- Also evicts the in-memory cache.

### Async wrappers
`LoadAllMacrosAsync()`, `SaveMacroAsync()`, `DeleteMacroAsync()` — `Task.Run` wrappers for UI responsiveness.

There is **no `LoadMacrosByProfile()`** — profile filtering is done in `config.json`, not SQLite.

## Schema Migration

`MacroDatabase.Initialize()` runs non-destructive migrations via `ALTER TABLE ... ADD COLUMN` wrapped in try/catch (no-op when the column already exists):

```csharp
// Example: adding Icon column if it doesn't exist
ALTER TABLE MacroSteps ADD COLUMN WindowTitle TEXT
```

This allows non-destructive schema evolution without migration frameworks. A pre-update backup (`macros_pre_update.bak`) is created before schema changes, and a corrupt DB is renamed to `.corrupt` and re-initialized fresh.

## Orphaned File Cleanup

`MacroDatabase` participates in media cleanup:
- `CleanupOrphanedFiles()` (renamed from the old `CollectImagePaths()` concept) wipes `TempImages`, then uses `GetReferencedMediaFiles()` to scan all `MacroSteps` (including `ExtraJson` `UIScreenshotPath` and `TraceFileId` entries) for referenced image/trace paths
- Unused images in the Images directory and `trace_*.dat` files in TraceData are deleted
- Stale capture-library entries (UI Elements, Images, Pixels, Windows) unused for 30 days are purged

## Thread Safety

- All database operations use a shared persistent connection with `Pooling=True;Cache=Shared` and a `lock(_dbLock)` guard across all methods
- WAL journal mode, `busy_timeout=5000`
- The `ConcurrentDictionary` in `ScriptManager` is unrelated (that's for AHK processes)

## Key Files

- [MacroDatabase.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Services/Managers/MacroDatabase.cs)

## Related Pages

- [[macro-item]]
- [[macro-editor]]
- [[script-library]]
- [[macro-transfer-manager]]
