---
tags: [manager, import, export, transfer, backup]
date: 2026-05-23
last_updated: 2026-08-01
sources:
  - PowerX.Services/Managers/MacroTransferManager.cs
status: current
---

# Macro Transfer Manager 📦

`MacroTransferManager` handles **import and export** of macros as `.pxmacro` packages. These are ZIP archives containing the macro JSON plus all referenced media files.

## Overview

- Export: macro → ZIP with JSON + images + traces + audio
- Import: ZIP → macro with media extraction + coordinate scaling
- Backup: bulk export/import of all macros at once
- Screen resolution scaling on import

## Package Format (`.pxmacro`)

A `.pxmacro` file is a standard ZIP archive containing:

```
macro.pxmacro (ZIP)
├── macro.json          ← MacroExportPackage JSON
├── Images/             ← Captured screenshots (PNG/BMP)
│   ├── capture_001.png
│   └── capture_002.png
├── TraceData/          ← Mouse trace recordings (CSV)
│   └── trace_abc.dat
└── Audio/              ← Custom sound files
    └── beep.wav
```

## Data Models

### `MacroExportPackage` (single macro)
```csharp
public class MacroExportPackage
{
    public double OriginalWidth { get; set; }   // Source screen width
    public double OriginalHeight { get; set; }  // Source screen height
    public MacroItem Macro { get; set; }
    public ActionItem HotkeyBinding { get; set; }  // Hotkey binding so the shortcut survives export/import
}
```

### `MacroBackupPackage` (bulk backup)
```csharp
public class MacroBackupPackage
{
    public double OriginalWidth { get; set; }
    public double OriginalHeight { get; set; }
    public List<MacroItem> Macros { get; set; }
    public List<ActionItem> HotkeyBindings { get; set; }  // F35 fix: hotkeys survive backup/restore
}
```

## Export Process

`ExportMacro(MacroItem macro, string destinationPath)`:

1. Create `MacroExportPackage` with current physical screen resolution (DPI-independent via `GetDeviceCaps`)
2. Serialize to JSON
3. `CollectMedia()` recursively scans all steps (including child branches) for:
   - Image paths (FindText captures)
   - Trace file paths (`.dat`)
   - Audio file paths (`.wav`, `.mp3`)
4. Create ZIP archive with `macro.json` + all media files
5. Files are organized into `Images/`, `TraceData/`, `Audio/` folders
6. Write to a `.tmp` file first, then atomically rename over the destination (BUG-A12 fix — original is never lost on failed write)

## Import Process

`ImportMacro(string sourcePath)`:

1. Open ZIP archive
2. Read and parse `macro.json`
3. Extract media files to their respective app directories:
   - Images → `%LOCALAPPDATA%/PowerXKeys/Engine/Images/`
   - Traces → `%APPDATA%/PowerX_Keys/TraceData/`
   - Audio → `%LOCALAPPDATA%/PowerXKeys/Engine/Audio/`
   - Entry names are sanitized (`Path.GetFileName`) to prevent ZIP path traversal attacks
4. **Coordinate Scaling**: If source screen resolution differs from current:
   ```
   scaleX = currentWidth / originalWidth
   scaleY = currentHeight / originalHeight
   newX = (int)(step.X * scaleX)
   newY = (int)(step.Y * scaleY)
   ```
5. Assign new `Guid`s to the macro and all steps (old→new ID map fixes `ParentId` links)
6. **Hotkey binding restore**: if the package has a `HotkeyBinding`, it is re-pointed to the new macro ID and re-saved to `ConfigManager`
7. Return the `MacroItem` for saving to database

## Media Collection

`CollectMedia()` recursively walks the step tree:
- Checks `ChildSteps` (If-true branches, Groups, Loops)
- Checks `ChildStepsFalse` (Else branches) — fixed in v5.0.1
- Collects unique paths via `HashSet<string>`
- Handles all media types: images, traces, audio

## File Paths

| Media Type | Storage Directory |
|-----------|-------------------|
| Images | `%LOCALAPPDATA%/PowerXKeys/Engine/Images/` |
| Traces | `%APPDATA%/PowerX_Keys/TraceData/` |
| Audio | `%LOCALAPPDATA%/PowerXKeys/Engine/Audio/` |

## Key Files

- [MacroTransferManager.cs](file:///c:/Users/Maaz/Documents/PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX.Services/Managers/MacroTransferManager.cs)

## Related Pages

- [[macro-item]]
- [[database-schema]]
- [[script-library]]
- [[mouse-trace]]
