---
tags: [feature, settings, dashboard, configuration]
date: 2026-08-01
sources:
  - ViewModels/SettingsDashboardViewModel.cs
  - Views/SettingsDashboardView.xaml
status: current
---

# Settings Dashboard ⚙️

The Settings Dashboard is the central configuration hub for PowerX Keys. It exposes 26+ global toggle settings organized into logical categories.

## Overview

- All settings persist via `ConfigManager` → `AppConfig.json`
- Changes trigger live engine reload (when Auto-Reload is enabled)
- Factory Reset support with full UI sync
- Version text in the settings update area is fed from the shared app version source, so it stays in sync with the rest of the app

## Settings Categories

### 🖱️ Mouse & Keyboard
- **Record Full Mouse Path**: Capture smooth mouse traces during recording
- **Mouse Coordinates**: Show real-time cursor position
- **Keyboard Input**: Enable keyboard capture in recordings
- **Extreme Paste Speed**: Sets the default mode for newly created Type Text blocks to clipboard Paste mode
- **Hardware Input Lock**: Block user input during macro execution

### ⚡ Engine
- **Auto-Start Engine**: Launch AHK engine on app startup
- **Auto-Reload Engine**: Recompile scripts automatically on changes
- **Undo History Depth**: Configurable undo levels — hard-capped at 100 (default 50)
- **Click Bundling Preset**: Threshold for smart click merging

### 🎨 Appearance / User Interface
- ~~**App UI Zoom**: Scale the interface (80%–110%)~~ — ⚠️ **Removed.** No such setting exists in the app.
- ~~**Performance Mode**: Disables GPU effects, forces software rendering~~ — ⚠️ **Removed.** No such setting exists in the app.
- **Show Accent Colors**: Toggle color-coded bars on the left edge of action blocks in the editor timeline.
- **Magnifier Settings**: Customize zoom level (5x–10x), pre-capture delay (0s–5s), smart box size (20px–200px), and color palette (smart capture, target highlight, pixel crosshair, float capture). Select magnifier style (Solid/Dashed) and shape (Square/Circle) with an interactive live checkerboard preview.

### 🤖 AI (PowerX Intelligence)
- **AI connection status**: Online/Offline badge
- **Daily quota tracking**: Generations remaining + time to reset
- **OpenRouter login/logout** via PKCE OAuth2
- ~~**Model selection**: Multiple AI models (Llama, Qwen, Gemma, etc.)~~ — ⚠️ **Removed.** No model selection UI exists; only `ActiveAIProvider` is stored in config, and actual model routing happens server-side via the AI proxy.

### 📱 Remote Server
- **Enable/disable** the embedded HTTP server
- **PIN configuration** for mobile authentication
- **Port setting** (default: 8745)

### 🏠 General
- **Minimize to System Tray**: Close button hides to tray
- **Humanize Timing**: Add random variation to delays
- **Smart Sensitivity**: Adjust gesture detection thresholds
- **Advanced Trigger Combos**: Allow mouse buttons (Middle, Side Buttons, combos) with modifiers as hotkeys, **and** keyboard combos — two non-modifier keys pressed simultaneously (e.g. `Tab+A`, `F1+1`, `~+B`) captured as `Key1 & Key2` AHK custom combinations (standalone Left/Right click triggers are still strictly blocked for safety)
- **Text Snippets Settings**: Global sound on snippet trigger toggle and case insensitivity toggles.

### 🛡️ Advanced & Security
- **VIP Admin Mode**: Restarts PowerX Keys with UAC elevation to automate protected games and applications. Designed with amber (`TokenOrange300`) branding.
- **Turbo Engine Mode**: Dynamically boosts engine priority while macros are actively running, then relaxes back to normal 3 seconds after the last step. Off by default — when off, behavior is identical to no boost. Designed with vibrant green (`TokenGreen300`) branding.
- **Master Kill Switch**: Instantly aborts all macro execution via a global hotkey (default: `Shift + Escape`). Styled in warning red (`TokenRed300`) to highlight its security-level function.

### ⌨️ Keyboard Manager (Beta)
- ~~**Visual Key Mapping**: A real-time virtual keyboard rendering showing which keys are bound to active macros~~ — ⚠️ **Removed.** No such component exists in the app.
- **Editor Shortcut Rebinding**: Rebind hotkeys for editor actions (Undo, Redo, Duplicate, Move Up/Down, insert steps, etc.) with automated conflict detection.
- **Show Insert Feedback**: Toggle neon visual glow animation when adding steps to the timeline.
- **Enable Window Picker Hotkeys**: Toggle window picker capture hotkeys (`Ctrl+D` and `Ctrl+P`).
- **Hide to Tray During Recording**: Automatically hides the main window to the system tray while recording is active.
- **Scrollbar Enabled**: Uses `SettingsCategoryScrollStyle` for layout compliance and scrolling on smaller viewports.

### ☕ Support the Developer ("Buy Me a Coffee")
> ⚠️ **Removed.** The Tip Jar UI (Ko-Fi-style support button and its notification sync) has been removed from the app — no `ko-fi`/`SupportDeveloper`/`TipJar` references remain in the codebase.

### 🔔 Premium Notification Toast System
- **Unified Frosted Glass Purple Theme**: All in-app notifications utilize a premium glassmorphic style with a `#CC120F1F` dark purple tinted background, `#A78BFA` borders, `#8B5CF6` shadow, and a springy animation (utilizing a `BackEase` easing function).
- **Dynamic Theming & Icons**: Under the hood, emojis (e.g., `✅`, `⚠️`, `❌`, `ℹ️`, `🔄`, etc.) are parsed to automatically determine the toast icon, foreground accent color, and header title, while stripping the emoji prefix from the visible text itself:
  - **Success (`✅`, `💾`, `🔌`, `🎯`)**: Checkmark icon (`&#xE73E;` or `&#xE74E;` or `&#xE7E9;` or `&#xE73A;`) with green `#34D399` accent.
  - **Warning (`⚠️`, `⏹`)**: Alert icon (`&#xE7BA;` or `&#xE71A;`) with yellow `#FBBF24` accent.
  - **Info (`ℹ️`, `ℹ`)**: Info icon `&#xE946;` with blue `#60A5FA` accent.
  - **Error (`❌`)**: Cross icon `&#xE711;` with red `#F87171` accent.
  - **Reset/Sync (`🔄`)**: Sync icon `&#xE72C;` with purple `#C084FC` accent.
  - **Item Removed (`🗑️`, `🗑`)**: Trash icon `&#xE74D;` with red `#F87171` accent.

## Factory Reset

The "Danger Zone" section provides:
- **Delete All Macros**: Wipes SQLite database + in-memory cache
- **Clear AI Data**: Removes stored API keys
- **Reset Settings**: Restores all 26+ toggles to defaults with live UI sync via `NotifyAllSettings()`

## Performance Mode

> ⚠️ **Removed.** The Performance Mode setting (software rendering via `RenderOptions.ProcessRenderMode = SoftwareOnly`, disabling GPU animations, etc.) no longer exists in the app. The section below is kept as historical reference.

Previously, when enabled:
- Forced `RenderOptions.ProcessRenderMode = SoftwareOnly`
- Disabled all GPU-accelerated animations and effects
- Persisted across restarts (loads on boot before UI renders)
- Stopped the AI pulse animation to prevent GPU drain
- Stopped all background loading spinners (engine, AI chat, timeline, and mobile tunnel) when inactive or minimized
- Collapsed step card blurs and overlays by default to avoid CPU-based software rendering overhead on low-spec systems

## Key Files

- [SettingsDashboardViewModel.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/ViewModels/SettingsDashboardViewModel.cs)
- [SettingsDashboardView.xaml](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/Views/SettingsDashboardView.xaml)
- [AppConfig.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Core/Models/AppConfig.cs)
- [ConfigManager.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Services/Services/ConfigManager.cs)

## Related Pages

- [[app-config]]
- [[ai-chat]]
- [[mobile-remote]]
- [[script-manager]]
