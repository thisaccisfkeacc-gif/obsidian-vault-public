---
tags: [service, search, files]
date: 2026-08-01
sources:
  - (none — historical)
status: removed
---

# File Search Service

> ⚠️ **Removed / Abandoned.** The Everything-based file search described below **never shipped** in PowerX Keys. It belonged to the abandoned **SmartMenuMockup** project, which no longer exists in the repository — there is no `FileSearchService.cs`, no `EverythingSDK.cs`, and no `Everything64.dll` anywhere in the codebase. The section below is kept purely as historical reference.

## Historical Design (from SmartMenuMockup)

**Summary (historical):** The `FileSearchService` handled background detection and silent startup of the Voidtools Everything engine, and processed file queries via P/Invoke thread-safely.

### Purpose
Enabled near-instantaneous file and folder searches across the system directly from the Smart Menu HTML interface.

### Key Methods
| Method | Purpose |
|--------|---------|
| `Initialize()` | Verified if `Everything.exe` is running, starting it silently from the local directory or program files path if needed. |
| `Search(string query)` | Took a string query, set search constraints, queried Everything, retrieved result names/paths/sizes/folders, and returned them as a serialized JSON array. |

### Dependencies
- Called `EverythingSDK.cs` (P/Invoke layer)
- Relied on `Everything64.dll`
- Integrated with `SmartMenuWebViewWindow.xaml.cs`

### How to Modify (historical)
- Any SDK calls in `Search()` were to be synchronized inside the `lock (_lock)` block.
- If modifying request flags, `Everything_SetRequestFlags()` had to be called before executing the query.

---

## What the app actually uses today

PowerX Keys does **not** perform system-wide file searching. The only "search" in the app filters **in-memory** collections loaded from the SQLite macros database — e.g. the Capture Library's search box matches `ElementName`, `Label`, `ColorHex`, `AppName`, etc. via `Contains` on the already-loaded entries, and the export macro picker filters macro names the same way. No OS-level file indexer is involved.

## Related Pages
- [[index]]
