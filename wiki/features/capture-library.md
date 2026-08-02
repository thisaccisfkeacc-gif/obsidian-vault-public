---
tags: [feature, ui, database]
date: 2026-08-01
sources:
  - C:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\PowerX.UI\Views\CaptureLibraryWindow.xaml
  - C:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\PowerX.UI\Views\CaptureLibraryWindow.xaml.cs
  - C:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\PowerX.Services\Managers\MacroDatabase.cs
  - C:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\PowerX.Core\Models\CaptureLibraryEntry.cs
status: completed
---

# 📂 Capture Library

**Summary:** The Capture Library allows users to save and reuse captured UI Elements, Images, and Pixel colors, preventing the need to re-capture assets repeatedly.

## Database Schema
The Capture Library persists data into four distinct SQLite tables:

### 1. `CaptureLibrary_UIElements`
Stores Windows UI Automation elements.
- `Id` (TEXT, PK): Unique Guid
- `ElementName` (TEXT): Custom or auto-generated name
- `AutomationId` (TEXT): UIA automation identifier
- `ClassName` (TEXT): Win32 window class name
- `ControlType` (TEXT): UIA element type (e.g. Button, ComboBox)
- `WindowTitle` (TEXT): Parent window title
- `ProcessName` (TEXT): Origin process name (added via migration in `MacroDatabase.cs`)
- `ElementPath` (TEXT): Tree route path
- `ScreenshotPath` (TEXT): Relative path to crop thumbnail
- `X`, `Y` (REAL): Coordinates
- `CapturedAt`, `LastUsedAt` (TEXT): Timestamps
- `UseCount` (INTEGER): Total count of selections
- `IsFavorite` (INTEGER): Pinned status (0 or 1)

### 2. `CaptureLibrary_Images`
Stores cropped FindText images.
- `Id` (TEXT, PK): Unique Guid
- `ImagePath` (TEXT): Location of the image file
- `Label` (TEXT): Custom user label
- `SourceApp` (TEXT): Origin window/app title
- `Width`, `Height` (INTEGER): Image dimensions
- `CapturedAt`, `LastUsedAt` (TEXT): Timestamps
- `UseCount` (INTEGER): Selection usage statistics
- `IsFavorite` (INTEGER): Pinned status (0 or 1)

### 3. `CaptureLibrary_Pixels`
Stores captured target pixel colors.
- `Id` (TEXT, PK): Unique Guid
- `ColorHex` (TEXT): Color hexadecimal value (e.g., `#288DE5`)
- `Label` (TEXT): Custom user label
- `SourceApp` (TEXT): Origin window/app title
- `X`, `Y` (REAL): Coordinates
- `CapturedAt`, `LastUsedAt` (TEXT): Timestamps
- `UseCount` (INTEGER): Selection usage statistics
- `IsFavorite` (INTEGER): Pinned status (0 or 1)

### 4. `CaptureLibrary_Windows`
Stores captured App Switcher target windows.
- `Id` (TEXT, PK): Unique Guid
- `AppName` (TEXT): Custom or app-derived name
- `WindowTitle` (TEXT): Captured window title
- `ProcessName` (TEXT): Origin process name (e.g. `chrome.exe`)
- `MatchMode` (TEXT, default `AnyWindow`): Window matching strategy
- `CapturedAt`, `LastUsedAt` (TEXT): Timestamps
- `UseCount` (INTEGER): Selection usage statistics
- `IsFavorite` (INTEGER): Pinned status (0 or 1)

---

## User Interface
The Capture Library is presented as a clean 4-tab popup dialog:
- **🪟 Elements Tab**: Lists all captured UI automation components with screenshot thumbnails, Window context, and favorite pins.
- **🖼️ Images Tab**: Lists all cropped visual search patterns.
- **🎨 Pixels Tab**: Shows all color indicators with swatches, hex codes, and copy-to-clipboard buttons.
- **🪟 Windows Tab**: Lists captured app/window targets for App Switcher steps, with process names and match modes.

### Key Workflows
1. **Auto-Save**: Successful captures in the Macro Editor automatically save to the database.
2. **Library Selection**: Clicking the `📂` (Library) button next to any capture control opens the library at the correct tab. Double-clicking or selecting an entry maps the properties back to the step.
3. **Stale Cleanup Banner**: A yellow warning banner alerts the user when captured assets are unused for 30+ days, offering a one-click purge action to clean database/files.
4. **Context Menus**: Right-click menu options are supported for renaming, deleting, copying, and toggling favorite pins on items.

---

## Related Pages
- [[database-schema]]
- [[macro-editor]]
- [[image-recognition]]
