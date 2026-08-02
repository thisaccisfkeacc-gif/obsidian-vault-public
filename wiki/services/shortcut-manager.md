---
tags: [service, shortcut-manager, hotkeys, input]
date: 2026-05-27
sources:
  - Services/ShortcutManager.cs
  - Views/MacroEditorView.xaml.cs
  - ViewModels/SettingsDashboardViewModel.cs
status: active
---

# Shortcut Manager ⌨️

The **Shortcut Manager** (`ShortcutManager`) is a static utility service that coordinates and manages customizable keyboard shortcuts within the Macro Editor. It handles default bindings, overlays user-defined configurations, detects conflicts, and matches incoming keyboard events to specific editor actions.

## Key Capabilities

- **Custom Bindings**: Allows rebinding of key combinations for common editing operations.
- **In-Memory & Disk Persistence**: Reads/writes custom mappings from `AppConfig.json` (via `ConfigManager.Current.Settings.EditorShortcuts`). To keep the JSON configuration clean, only customized shortcuts (non-default) are written to disk.
- **Conflict Detection**: Checks if a key+modifier combination is already bound to another action before applying changes.
- **WPF Gesture Integration**: Uses WPF's `System.Windows.Input.Key` and `ModifierKeys` to capture key inputs.

## Editor Actions & Defaults

The service manages the `EditorAction` enum containing 9 actions across two categories:

| Action (`EditorAction`) | Default Gesture | Category | Description |
|-------------------------|-----------------|----------|-------------|
| `Undo` | `Ctrl+Z` | Editing | Revert the last editor state snapshot |
| `Redo` | `Ctrl+Y` | Editing | Re-apply a reverted editor state |
| `MoveUp` | `Ctrl+↑` | Editing | Move the selected macro step up in the timeline |
| `MoveDown` | `Ctrl+↓` | Editing | Move the selected macro step down in the timeline |
| `Duplicate` | `Ctrl+D` | Editing | Duplicate the currently selected block |
| `AddKeyboard` | `Ctrl+K` | Quick Add | Append a new Keyboard action step |
| `AddMouse` | `Ctrl+M` | Quick Add | Append a new Mouse action step |
| `AddWait` | `Ctrl+W` | Quick Add | Append a new Delay / Wait step |
| `AddText` | `Ctrl+T` | Quick Add | Append a new Type Text step |

## Key Methods

| Method | Return Type | Description |
|--------|-------------|-------------|
| `Initialize()` | `void` | Loads hardcoded defaults and overlays saved configurations from settings |
| `TryMatch(KeyEventArgs e)` | `EditorAction?` | Matches a WPF key event to a registered action |
| `Rebind(EditorAction, Key, ModifierKeys)` | `void` | Updates an action's shortcut, triggers auto-saving to config |
| `FindConflict(Key, ModifierKeys, excludeAction)` | `EditorAction?` | Returns any action that already uses the key/modifiers combination |
| `ResetToDefaults()` | `void` | Wipes custom shortcuts and restores factory default configurations |

## Usage in Macro Editor

In [MacroEditorView.xaml.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/MacroEditorView.xaml.cs), the `PreviewKeyDown` handler intercepts keystrokes and queries the `ShortcutManager`:

```csharp
private void MacroEditor_PreviewKeyDown(object sender, KeyEventArgs e)
{
    // Check if the user is in a text box to avoid blocking standard text input
    if (IsUserEditingText()) return;

    var action = ShortcutManager.TryMatch(e);
    if (action.HasValue)
    {
        e.Handled = true;
        ExecuteEditorAction(action.Value);
    }
}
```

## Key Files

- [ShortcutManager.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Services/ShortcutManager.cs) — contains the binding, parsing, and management logic.

## Related Pages

- [[macro-editor]]
- [[settings-dashboard]]
- [[app-config]]
