---
tags: [feature, macro-editor, ui, viewmodel]
date: 2026-08-01
sources:
  - Views/MacroEditorView.xaml.cs
  - ViewModels/MacroEditorViewModel.cs
status: current
---

# Macro Editor 🎬

The Macro Editor is the primary workspace where users build, record, and edit automation macros. It presents a **timeline-style UI** where each action is a visual card.

## Overview

- Users can **record** macros (mouse/keyboard capture) or **manually add** steps from a toolbox
- Steps are displayed as cards in a vertical timeline with drag-and-drop reordering
- Supports nested logic blocks (If/Else, Loops, Groups)
- Two view modes: **Timeline** (full cards) and **Extreme Mini View** (compact DataGrid)

## Architecture

Both the View and ViewModel layers are modularized using **partial class** and custom sub-control architectures for maintainability and UI performance:

### View Architecture

The View layer is split into multiple files to improve IDE designer loading times and keep code organized:

| Component / File | Type | Responsibility |
|------------------|------|----------------|
| [MacroEditorView.xaml](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/Views/MacroEditorView.xaml) | View | Main editor shell layout containing toolbar, timeline lists, and subviews |
| [MacroEditorView.xaml.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/Views/MacroEditorView.xaml.cs) | Code-behind | Main partial class hosting constructor, Loaded/Unloaded lifecycle hooks, and drag-drop fields |
| [MacroEditorView.Events.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/Views/MacroEditorView.Events.cs) | Code-behind | Partial class hosting keyboard/mouse events, shortcuts (ESC, etc.), save click routing, and validation |
| [MacroEditorView.DragDrop.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/Views/MacroEditorView.DragDrop.cs) | Code-behind | Partial class hosting timeline ListBox/DataGrid dragging, custom lasso selection, and drop indicator logic |
| [MacroEditorCheatSheet.xaml](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/Views/MacroEditorCheatSheet.xaml) | Sub-view | Reusable `UserControl` containing F1 shortcut guide list and layout |
| [MacroEditorOverlays.xaml](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/Views/MacroEditorOverlays.xaml) | Sub-view | Reusable `UserControl` containing dialog popups (`SaveDialog`, `WarningDialog`, `HumanizationWarningDialog`) and `ModernToast` notification |

### ViewModel Architecture

The ViewModel is split into 8 partial C# class files:

| File | Responsibility |
|------|---------------|
| `MacroEditorViewModel.cs` | Constructor, initialization |
| `Core.cs` | Step management, save/load, navigation guards |
| `Properties.cs` | All bindable properties |
| `Commands.cs` | ICommand definitions (Save, Delete, Add Step, etc.) |
| `Recording.cs` | Mouse/keyboard recording engine integration |
| `Capture.cs` | Screen capture overlay for image/pixel targets |
| `Optimization.cs` | Smart bundling, loop detection, step cleanup |
| `SmartView.cs` | Smart View bundling, delay merging, and timeline diff-sync |
## Key Capabilities

### Recording Engine
- Launches an AHK recording script via `ScriptManager`
- Captures raw keystrokes + mouse events at ~20ms intervals
- Supports **resuming/inserting recording from a selected step** (or appending to the end if no step is selected)
- Includes a **recording position safety confirmation popup** when a step is selected, preventing accidental step insertion in the middle of a macro
- Post-processes with `OptimizeRecordedSteps()`:
  - Cleans up, merges, and trims delays and redundant elements
  - Preserves raw keyboard hold/release steps to keep the timeline raw-editable
- Preserves `IsManuallyAdded` steps from cleanup

### Step Types (`MacroStepType` enum)
- Mouse actions: Click, Drag, Scroll, Hold, Release
- Keyboard: Key press, Type Text, Wait for Key
- Timing: Wait/Delay
- Logic: If/Else, Loop, Group
- Visual: ImageSearch, PixelSearch, FindText
- Advanced: Mouse Trace, File Launcher, System Sound, Notification, User Input

### Drag & Drop
- Custom drag-and-drop system with a simple thin blue line indicator
- Prevents circular references (moving parent into own child)
- Supports cross-nesting into logic blocks
- Undo snapshot captured before every drag operation

### Undo/Redo
- Snapshot-based system for editor state rollback
- Configurable depth, hard-capped at 100 levels (default 50)
- Single snapshot for bulk operations (multi-delete, drag-reorder)

### Customizable Editor Shortcuts
- Allows binding custom hotkeys for common editor tasks:
  - **Undo / Redo** (Defaults: `Ctrl+Z` / `Ctrl+Y`)
  - **Move Step Up / Down** (Defaults: `Ctrl+Up` / `Ctrl+Down`)
  - **Duplicate Step** (Default: `Ctrl+D`)
  - **Add Keyboard / Mouse / Wait / Text Step** (Defaults: `Ctrl+K` / `Ctrl+M` / `Ctrl+W` / `Ctrl+T`)
- Custom keys are configured globally in the settings dashboard and enforced via the `ShortcutManager` service

### Preview Speed Controls
- Located in the macro options dropdown menu:
  - **Fast Scroll**: Bypasses recorded delays between mouse scrolls during timeline preview
  - **Fast Click**: Bypasses recorded delays between mouse clicks during timeline preview
  - **Fast Typing**: Bypasses recorded delays between keystrokes during timeline preview

### Add Action Menu
- **Collapsible Section Headers**: All 4 categories (Basic Actions, Logic, System/Advanced, Templates) independently collapse/expand via clickable headers with chevron toggles. Rounded corners adapt when collapsed.
- **View Cycling Toggle**: Cycle between Detailed (icon + title + description) and Compact (icon + title) view modes.
- **Persistent State**: The collapse states and selected layout persist across app restarts using config properties.

### Quick Action Overlay Capture
- **App Switcher Quick Capture**: When an App Switcher step (type `WindowAction`) is newly added and does not yet have a target window set (i.e. `WindowTitle` is empty), clicking directly on the step's timeline container card will automatically trigger the window capture utility (`CaptureWindowCommand`).
- **Window Pinpoint Capture**: The Window block's `Pinpoint` overlay now finalizes the highlighted window on left click by resolving its process name and window title before closing, so the block target field updates immediately.
- **Manual Coordinate Editing**: The Mouse block's `Edit Coordinates` dialog now blocks malformed numeric input while typing, shows clearer X/Y save errors, and clamps saved values to the virtual screen bounds.
- **Interaction Rules**:
  - Once a window is captured, clicking the card container performs normal selection and editing.
  - Recapturing a window can be triggered via the settings gear icon options.
  - Interactive inputs (combo boxes, textboxes, buttons, toggle buttons, popups) ignore the outer container click-to-capture trigger.

## Visual Design & Timeline Card Styling 🎨

Timeline cards follow a **segmented inline** design pattern — a sleek, minimal, horizontal layout inspired by modern IDE toolbars ([MacroStepCard.xaml](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/Views/MacroStepCard.xaml) & [MacroStepTemplates.xaml](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/Views/MacroStepTemplates.xaml)):
- **Segmented Inline Layout**: Each block is a horizontal `StackPanel` with controls separated by thin vertical dividers (`1px #2D2F36 borders`).
  - Example for Mouse: `[Left Click ▾] │ [▶ Press] │ at [520, 340] [📍] │ [☉] │ [⚙]`
  - Example for UI Element: `[Name] │ [Click ▾] │ Select [📍] │ [⚙] │ [Button] Element Name` (captured element badge only appears when set, preceded by a dynamic divider)
- **No Accent Bars**: The old left-side gradient accent bars (`<Border Width="3" Collapsed>`) and their `<Grid>` wrappers have been fully removed from all 20 templates for a cleaner, flatter look.
- **Dark Pill Controls**: Interactive elements use `#1C1D21` background with `#2D2F36` borders and `CornerRadius="6"`. Hover state: `#25262B` bg + `#5C5E6B` border.
- **Tri-State Cycling Buttons**: Press/Hold/Release buttons cycle on click with icon + label (`CornerRadius="6"`).
- **Settings Gear Popups**: A ⚙ ToggleButton opens a Popup with advanced settings (humanization, coordinate mode, etc.). Gear icon turns red when humanization is active. All 14 settings popups have been standardized for consistency:
  - **Headers**: UPPERCASE, Bold, `#7F828E` foreground
  - **Info icons**: Base `#7F828E`, hover `#C0C2C8`
  - **Toggles**: Universal blue `#5AC8FA` when checked, 28×28 size
  - **Dropdowns**: All use `DarkParamComboBoxStyle`
  - **Checkboxes**: All use `DropdownMenuCheckBoxStyle`
  - **Container padding**: Consistent `10px`
  - **Emojis removed**: All emoji labels replaced with clean text
  - Redundant pinpoint capture buttons removed for cleaner layout.
- **Vertical Dividers**: `1px × 18px` `#2D2F36` borders visually separate logical segments (action, position, settings).
- **Dynamic Dividers**: Certain vertical dividers (like the ones before the mouse/keyboard Press/Hold/Release toggle or the UI Element captured details) dynamically collapse when their adjacent target element is hidden/empty, keeping the block clean.
- **Purple Pinpoint & Visible Coordinate Pill**: The pinpoint capture button `[☉]` on the main timeline row is highlighted in a premium purple (`#A78BFA`) that lights up on hover. The coordinate display pill also uses the same premium vector pinpoint icon (`[☉]`) and stays visible even when values are empty, showing as `☉ X, Y` as a clean placeholder so the user can easily see where to input or click to capture coordinates.
- **Uniform Button Sizing**: All settings gear buttons (`CoordsHumanizeToggle`, `KeyHumanizeToggle`, and `DelayHumanizeToggle`) across different timeline block panels have been standardized to a uniform size (`Width="28" Height="28"`) to match the visual footprint of the pinpoint capture button and maintain styling alignment.
- **Sleek Inputs and Placeholders**: Placeholder texts like "+ Empty" in the Type Text block are styled in a neutral gray (`#7F828E`) when no value has been entered, and switch to a bold blue (`#5AC8FA`) only when text is present. Preset dropdown menus and plus/minus buttons (like the inline `+`/`-` duration adjustments in the Wait/Delay block) use the standard Sleek dark gray background (`#1C1D21`) and gray border (`#3B3D46`) to blend in seamlessly with the dark mode aesthetic.
- **Sleek Camera, Color & Search Icons (Find Image / Find Color)**: The Capture Pattern button in Find Image uses a sleek, vector-based camera icon (`&#xE722;`, `FontSize="12"`) which stays in a neutral gray `#8E909A` when no image is captured, turning a vibrant green `#4ADE80` only when an image is captured. The orange search emoji `🔍` in both Find Image and Find Color (PixelSearch) templates has been replaced with a modern, thin vector magnifying glass search icon (`&#xE721;`, `FontSize="12"`, `#C0C2C8`). Spacing, margins, and settings gear backgrounds across both templates have been standardized to a uniform `6px` layout for a tighter, cohesive horizontal alignment.
- **Standardized Header Icon Sizes**: All 22 timeline step card types have been configured with custom `FontSize` settings (ranging from `13.5` for bulky solid icons like the color palette or folder, to `17.5` for thin line icons like mouse trace or branching) to visually normalize their visual weights. In Compact Mode, these visual proportions are preserved dynamically using a `ScaleTransform` at 80% scale rather than being overridden by a single hardcoded size, keeping the layout sleek and consistent.


### Collapsible Block v2 — Inline Compact Design

All collapsible/nestable blocks (Group, If/Else, Repeat) use a unified **single-row inline** layout. Their config panels render in the **header row** (Row 0) alongside the title — not below it. This eliminates wasted vertical space.

| Block | Layout | Rename |
|-------|--------|--------|
| **Group** | `▼ Group 1  [3 blocks]` | Double-click name to edit |
| **If/Else** | `▼ [Above Step Succeeded ▾] ⇅` | N/A (dropdown) |
| **Repeat** | `▼ Repeat 1  [3] ×` | Double-click name to edit |

**Key design decisions:**
- **No color dots/borders on Groups** — Removed all `GroupColorBrush`, `GroupBorderColor`, `GroupBackgroundColor` references. Group text is neutral white (`#E1E3E8`).
- **No redundant labels** — Removed `"Condition:"` from If/Else, `"Name:"` and `"Repeat:"` from Repeat block. The controls are self-explanatory.
- **Auto-incremented naming** — All named blocks get auto-numbered names on creation:
  - Groups use `Value` property: `"Group 1"`, `"Group 2"` via `GetNextBlockName()`
  - Loops use `Value` property: `"Repeat 1"`, `"Repeat 2"` via `GetNextBlockName()`
  - ImageSearch, PixelSearch, UIElement, WaitUntil, CallMacro, WindowAction use `StepName` property: `"Find Image 1"`, `"Find Color 1"`, `"UI Element 1"`, `"Wait Until 1"`, `"Call Macro 1"`, `"Window 1"` via `GetNextStepName()`
  - The numbering scans recursively through all steps (including nested children) and increments from the highest existing number.
- **Double-click-to-rename** — All named blocks (Group, Repeat, Image, Pixel, UI Element, Wait Until) use `IsEditingProperties` toggle: `TextBlock` (label mode) ↔ `TextBox` (edit mode). Enter/Escape/LostFocus all dismiss the editor. The `GroupNameEdit_LostFocus` handler in `MacroStepTemplates.xaml.cs` sets `IsEditingProperties = false` when clicking outside.
- **Block count badge** — Groups show a `[N blocks]` badge when collapsed, hidden when expanded or empty.
- **Inline panels** — Loaded via `ContentControl` placeholders (`GroupPanelInline`, `LogicIfPanelInline`, `LoopPanelInline`) in [MacroStepCard.xaml](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/Views/MacroStepCard.xaml) Row 0.
- **Sleek UI Element Block** — Moved captured element info (type badge and name) inline rather than rendering a separate row below, separated by a dynamic vertical divider that automatically hides when no element is captured. Changed action dropdown to use `DarkParamComboBoxStyle` for consistency.

## View Modes

- **Smart View**: Dynamically bundles and groups raw keyboard holds/releases and characters into human-readable blocks (e.g., "Shortcut: CTRL + C", "Type Text") using in-memory wrap and bundling engines. Also bundles mouse actions: Hold Down → Release pairs are merged into "Left Click", two rapid clicks into "Double Click", and 3+ clicks into a single "Multiple Clicks" block with adjustable ClickCount via +/- buttons. Click bundling tolerance is **≤5px for the first click pair** and **≤10px for subsequent clicks**. Internal shortcut delays (typing latency < 800ms) are absorbed into VirtualSourceSteps, and small delays (< 800ms) between consecutive keyboard shortcuts are consumed in post-processing to reduce timeline clutter. Edits made to virtual text or virtual delay steps dynamically propagate back to the underlying raw steps and are fully supported by the undo/redo system (via state snapshotting before modification).
- **Raw View**: Shows every individual raw Hold/Release/Movement step exactly as recorded, allowing granular editing of key state sequences.
- **Extreme Mini View**: Compact DataGrid with accordion expand, keyboard navigation, context menus

### Timeline Layout Modes (3-Mode System)

The editor supports three layout modes cycled via the layout button (`CycleTimelineLayoutCommand` / `TimelineLayoutIndex`):

| Mode | Index | Icon Color | Properties | Editing |
|------|-------|------------|------------|---------|
| **Normal** | 0 | Gray | Full size, all visible | Direct |
| **Compact** | 1 | Yellow | Hidden, summary shown | Double-click to expand |
| **List** | 2 | Red | Hidden, summary shown | Double-click to expand |

**Compact (1)** — Tighter card spacing: 24×24 icon (0.75x scale), 11px title font, 2px card gap, hover-only delete button. ConfigContainer collapsed by default, expanded via `IsExpandedInCompactMode` toggle on double-click.

**List (2)** — Ultra-minimal: 22×22 icon (0.7x scale), 11px title font, 1px card gap. Same collapse/expand behavior as Compact but with even less spacing.

**Summary Text (`DisplayValue`)** — When ConfigContainer is collapsed, a `SummaryContainer` shows contextual preview text for every block type:
- Mouse: `Left Click`, `Hold Down`, `Scroll Up (x5)`
- Keyboard: `Press A`, `Hold Ctrl`, `Release Shift`
- Image: StepName (e.g. `ButtonIcon`) or `No image captured`
- Pixel: StepName or TargetColorHex or `No pixel captured`
- UI Element: `Click · Submit Button` (action + element name)
- Popup: PopupMode (e.g. `Checkpoint`)
- Notification: Value text or `Empty notification`
- Wait Key: `Enter / Escape` (continue/cancel keys)
- User Input: `Text input`, `Dropdown input`
- File Launcher: filename only (e.g. `chrome.exe`)
- Sound: SoundType (e.g. `Default Sound`)
- Wait Until: `Wait Image: ButtonIcon`, `Wait Pixel: #FF0000`
- Call Macro: macro name

**Accent Colors** — Optional color-coded left edge bars per block type, toggled via `ShowAccentBars` in Editor Settings → View & Display. Each of the 22 step types has a unique color (e.g. blue for mouse, green for keyboard, orange for logic). Persists across restarts via `ConfigManager`. The accent bar also appears on selected steps regardless of the toggle state.

## High-Performance Timeline Reconciliation ⚡

- **Incremental Diffing (`SyncTo`):** Uses an in-place collection synchronization algorithm mapping target display steps to current steps via `Move(oldIdx, newIdx)` and `Insert(newIdx)` rather than clearing and reloading. This preserves WPF's ListBox visual containers (`ListBoxItem`) and prevents CPU-heavy visual tree regeneration.
  - **Surgical ID Preservation:** Raw steps are matched by their unique `Id` (instead of reference identity) inside the `AreStepsEqual` comparison method.
  - **Undo/Redo ID Continuity:** During undo/redo pushes, step IDs are preserved via `Clone(preserveId: true)`.
  - **Redundant Event Prevention:** The `ParentId` property features an equality guard, preventing WPF from receiving thousands of redundant layout notifications during recursive refreshes.
  - **$O(N)$ Synchronization:** Replaced the $O(N^2)$ SyncTo synchronization loop with a fast $O(N)$ removal pass using a `LambdaEqualityComparer` and safe hash codes calculated from step properties.
  - **Property Change Guards:** Added guards (`if (field != value)`) on properties modified during syncing (`Depth`, `StepName`, `Duration`, `ClickCount`) to avoid raising redundant property change events.
  - **Conditional CollectionView Refresh:** Modified `RunRefreshDisplayStepsAsync` to only trigger `DisplayMacroSteps.Refresh()` when the visibility filter itself changes (`IsDelayHidden`), letting WPF handle item updates incrementally and instantly.
  Together, these updates ensure reordering, adding, deleting, and undo/redo operations run instantly and lag-free (under 10ms), even with 500+ steps.
- **Snap to Block & Aurora Pulse UX:** When a step is newly inserted, duplicated, or recorded, the editor scrolls to the step (`ScrollIntoView`) and triggers a premium **Aurora Pulse** feedback effect. This effect consists of a blurred gradient backlight (`#8B5CF6` ➔ `#A78BFA` ➔ `#EC4899`) that scales and pulses behind the card, combined with a precise purple overlay border that fades in/out gracefully using storyboard animations over 1.5 seconds.
- **Timeline Loading Overlay:** When a recording stops or background layout processing runs, `IsTimelineProcessing` is toggled. If the timeline is empty, a modern rotating glassmorphism loader shows ("Processing Recording...") to give immediate visual feedback, while hiding the default "+ Drag to add step..." empty placeholder.

## Navigation Guard

The editor tracks an `IsDirty` flag. Navigating away or attempting to close the application triggers an unsaved changes warning dialog (if `SafetyConfirmations` is enabled in settings). 

The dialog features:
- **EXIT**: Discards changes and navigates away or closes the app (if triggered by a window close event).
- **CANCEL**: Keeps the user in the editor and prevents navigation or application exit.

The tray-minimize feature and direct tray-exit bypass this guard to ensure a smooth tray experience.

## Key Files

- [MacroEditorViewModel.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/ViewModels/MacroEditorViewModel.cs)
- [MacroEditorView.xaml](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/Views/MacroEditorView.xaml)
- [MacroEditorView.xaml.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/Views/MacroEditorView.xaml.cs)
- [MacroEditorView.Events.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/Views/MacroEditorView.Events.cs)
- [MacroEditorView.DragDrop.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/Views/MacroEditorView.DragDrop.cs)
- [MacroEditorCheatSheet.xaml](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/Views/MacroEditorCheatSheet.xaml)
- [MacroEditorOverlays.xaml](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/Views/MacroEditorOverlays.xaml)
- [MacroStepCard.xaml](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/Views/MacroStepCard.xaml)
- [MacroEditorViewModel.SmartView.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/ViewModels/MacroEditorViewModel.SmartView.cs) — Smart View bundling engine

## Related Pages

- [[macro-item]]
- [[execution-pipeline]]
- [[mouse-trace]]
- [[image-recognition]]
- [[script-library]]
