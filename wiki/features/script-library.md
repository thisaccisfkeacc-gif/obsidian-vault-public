---
tags: [feature, script-library, dashboard, hotkeys]
date: 2026-08-01
sources:
  - ViewModels/ScriptLibraryViewModel.cs
  - ViewModels/ScriptLibraryViewModel.Commands.cs
  - ViewModels/ScriptLibraryViewModel.State.cs
  - Views/ScriptLibraryView.xaml
  - Views/CustomActionCard.xaml
status: current
---

# Script Library (Main Dashboard) 📚

The Script Library is the **home screen** of PowerX Keys. It displays all saved macros, manages hotkey bindings, and controls the AHK engine state.

## Overview

- Three-tier sidebar: **Profiles** → **Macros** → **Editor**
- Global engine toggle (master ON/OFF switch)
- Per-macro enable/disable toggles with conflict detection

## Architecture

`ScriptLibraryViewModel` is the central dashboard ViewModel. It has been modularized into three partial class files for readability and maintainability:
- **`ScriptLibraryViewModel.cs`**: Contains core properties, constructors, data loading, and Dispose logic.
- **`ScriptLibraryViewModel.Commands.cs`**: Handles all UI RelayCommands and action execution (including interop window captures).
- **`ScriptLibraryViewModel.State.cs`**: Contains property-changed listeners, validation check routines, and timer-driven auto-saves and engine updates.

It:
- Loads all macros from `MacroDatabase.LoadAllMacros()`
- Reads hotkey bindings from `ConfigManager.Current.Hotkeys`
- Syncs enable/disable state between UI toggles and the AHK engine
- Triggers engine reload when bindings change

## Key Features

### Hotkey Binding System
- Each macro is bound via an `ActionItem` in the config
- Supported trigger modes: Press, Double Tap, Long Press, Hold Down, Release Up, Toggle (N-slot), Auto-Detect (screen event), Mobile Remote
  - **Toggle Mode**: Allows a single hotkey to cycle through up to 5 different macro slots (A, B, C, D, E) sequentially. Each card maintains its own toggle sequence, which can be fully edited using the dropdown selectors inside the card's settings gear popup.
- **Hotkey Safety Block**: Standalone mouse Left Click (`{LButton}`) and Right Click (`{RButton}`) are strictly blocked from being bound as trigger hotkeys (both on the dashboard card listening mode and inside the macro editor key capture dialog) to prevent accidental mouse lockout and preserve system navigation. If attempted, the binding is canceled and a warning/toast is shown to the user.
- Conflict detection highlights duplicate bindings with red borders

### Window Cycling Behavior
- App Switcher can still cycle through multiple open windows of the same app when that option is enabled.
- When strict matching is used, the captured app name plus window title are used together so the same window can still be found more reliably after reopen or restart.
- Restore Size & Position uses the saved window bounds from the latest capture.
- Turning on Restore Size & Position reveals the related window targeting settings in the gear menu.


### App Scoping (Targeting)
- Supported for both **Custom Macros** and **Custom Actions** (File Launchers, Custom Keystrokes, App Switcher).
- Accessible via the **Settings Gear Icon ⚙️** on each card.
- Scoping modes include:
  - **Active Everywhere** (Global): Hotkey triggers globally on the system.
  - **Include Only**: Hotkey is active only when one of the specified target windows is active.
  - **Exclude Target**: Hotkey is disabled when any of the specified target windows is active.
- Includes interactive **Capture Target App** countdown and clear options.
- Custom actions display active scoping tags (e.g. "Included: notepad.exe" or "Excluded: chrome.exe") directly on their cards for fast visual feedback.

### Unified ON/OFF Toggle Switches
- Sleek, pill-shaped toggle switches (`CompactToggleSwitchStyle`) are used globally.
- Displays **"ON"** in bold white text with a blue background when active, and **"OFF"** in gray text with a dark background when inactive.
- Applied to all dashboard items, including:
  - **Custom Macro Cards**: Replaced the circular green buttons.
  - **Default Action Cards**: Added enable/disable switches to allow temporary deactivation.
  - **Text Snippets**: Reused for consistent styling.
- Toggling automatically validates hotkeys (preventing enabling custom macros without hotkeys), triggers live engine reloads if active, and displays user status toasts.
- **Default/Startup Behavior**: On a fresh install, all default and custom action sub-switches that have assigned hotkeys are **ON** by default (those without assigned hotkeys are OFF). Toggled states are preserved across app restarts.

### Engine Synchronization
- Master toggle controls global engine state
- Turning OFF the master switch disables all individual macro toggles
- Auto-reload setting: changes can apply instantly or require manual restart
- Uses `ScriptManager.Stop()` → `ScriptCompilerService.CompileMasterScript()` → `ScriptManager.Start()`
- **Dynamic Conflict Validation**: Conflict calculations are re-run on demand when the engine's active profile or running profile states change (via `RunningProfiles.CollectionChanged` handler in `MainViewModel` and inside `RefreshLibrary` when switching profiles), ensuring live feedback.
- **Global Start Button Context**: The title-bar START button contextually disables if no hotkeys are active or if there are conflicts. It queries the underlying `ConfigManager.Current.Hotkeys` rather than the active view model to remain interactive even when the user navigates away (e.g., to the Settings view).

### Profile System
- Macros belong to profiles (e.g., "Default", "Gaming", "Work")
- Profile switching reloads the macro list
- Duplicate/reserved profile name validation
- Custom profile creation dialog (`ProfileCreationWindow`) features a grid of vibrant color emoji preset options, and allows entering any custom emoji via a dedicated text input and Windows emoji panel.
- The sidebar profile list dynamically renders color emojis (like `🏠`, `🚀`, `⚡`, `📁`, `🎮`) in high-fidelity color for both system and custom profiles.

### Dashboard Categories
- **My Macros**: User-created macros with hotkey bindings (formerly "Custom Macros")
- **File Launchers**: Quick-launch apps/files
- **Custom Keystrokes**: Simulate key combinations
- **Media Controls**: Volume adjusting and mute actions (Volume Up, Volume Down, Volume Set 50, Mute)
- **Media Playback**: Media navigation controls (Play / Pause, Next Track, Previous Track)
- **Browser Tabs**: Quick tab switching (Prev Tab, Next Tab) — also includes page history navigation (Go Back, Go Forward)
- ~~**Browser Navigation**~~ — legacy category; it only exists as a cleanup migration in `ConfigManager` (old "Browser Navigation" entries are removed on load). Go Back / Go Forward now live under **Browser Tabs**.
- **System Navigation**: Virtual desktop switching (Desktop Left, Desktop Right)
- **Quick Actions**: Handy utilities (Lock PC, Screenshot, Show Desktop) (formerly "Custom Actions")

### Macro Management
- Create, edit, duplicate, delete macros
- Double-click on any macro card to open it directly in the timeline editor
- Import/export via `MacroTransferManager`
- Redesigned, sleek emoji icon selector popup (featuring an obsidian theme, soft drop shadows, and modern hover borders) with native full-color emoji input via `emoji:RichTextBox` for visual identification.
- Cards feature refined, toned-down purple hover border outlines (`#7C3AED`) and soft drop shadow glows (`Opacity="0.15"`, `BlurRadius="8"`) for a premium and non-intrusive interactive experience.

### Double-Click to Edit & Simple Descriptions (Custom Actions)
- **Inline Name Editing**: Custom action card names are editable. By default, they act like static labels with a standard pointer cursor and a tooltip ("Double-click to edit name"). Double-clicking a card name unlocks it, changes the cursor to an I-beam, focuses it, and selects all text for editing. Pressing `Enter` or losing focus saves the changes; pressing `Escape` cancels them.
- **Default Simple Descriptions**: Custom actions feature user-friendly, non-technical description lines on their cards to match the aesthetic of built-in action cards:
  - **File Launchers**: *"Opens your selected app, file, or website."*
  - **Custom Keystrokes**: *"Presses your chosen keyboard keys."*
  - **App Switcher**: *"Quickly switches to your chosen app."*
- **App Switcher Matching**: Strict App Switcher targeting now follows the captured app name and window title together instead of relying on a saved old window handle.
- **File Launcher URL Input**: The File Launcher gear popup keeps File / URL Settings at the top, followed by the Which App section. Plain web addresses like `google.com` or `www.google.com` are accepted, and the Browse button can select a local file.
- **Unified List Layout & Type Icons**: Category section headers for File Launchers, Custom Keystrokes, and App Switcher are removed to create a clean, spacing-optimized, continuous list. Each card displays a styled, color-matched border icon at the start (Column 0) to clearly distinguish between File Launcher (blue icon), Custom Keystroke (yellow icon), and App Switcher (green icon) types.

## Visual Keyboard

> ⚠️ **Removed.** The interactive virtual keyboard visualization (showing bound hotkeys mapped to physical keys, dynamically updating with modifiers) no longer exists in the app — no such component is present in the codebase.

## Key Files

- [ScriptLibraryViewModel.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/ViewModels/ScriptLibraryViewModel.cs)
- [ScriptLibraryViewModel.Commands.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/ViewModels/ScriptLibraryViewModel.Commands.cs)
- [ScriptLibraryViewModel.State.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/ViewModels/ScriptLibraryViewModel.State.cs)
- [ScriptLibraryView.xaml](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/Views/ScriptLibraryView.xaml)
- [CustomActionCard.xaml](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/Views/CustomActionCard.xaml) — Card UI template displaying custom macro items

## Related Pages

- [[macro-editor]]
- [[app-config]]
- [[script-manager]]
- [[macro-transfer-manager]]
