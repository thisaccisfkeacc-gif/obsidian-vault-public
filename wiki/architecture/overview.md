---
tags:
  - architecture
  - overview
  - tech-stack
date: 2026-05-23
last_updated: 2026-08-01
sources:
  - PowerX.Core/Models/VersionInfo.cs
  - App.xaml.cs
status: complete
---

# Architecture Overview

PowerX Keys is an **advanced macro automation suite** for Windows built with C# .NET 10.0. It uses a unique **dual-execution architecture** — combining a WPF desktop GUI with a white-labeled AutoHotkey v2 backend engine.

## Tech Stack

| Layer | Technology |
|---|---|
| **Framework** | .NET 10.0 (`net10.0-windows`) |
| **UI** | WPF (primary) + WinForms (secondary — `SendKeys`, system tray) |
| **Scripting Engine** | AutoHotkey v2 (via `PowerX_Engine.exe`, a white-labeled `AutoHotkey64.exe`) |
| **Database** | SQLite via `Microsoft.Data.Sqlite 10.0.6` |
| **AI** | Server-side AI via Supabase Edge Function proxy (`AIFallbackService`); login via `SupabaseAuthService` (email OTP / Google OAuth) |
| **Remote Control** | Embedded HTTP server with mobile web UI |
| **NuGet Packages** | `Emoji.Wpf 0.3.4`, `QRCoder 1.8.0` |

## Version

- **App Version**: v5.4.0 (defined in [VersionInfo.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Core/Models/VersionInfo.cs))
- **Product**: "PowerX Keys" by PowerX Industries
- **Description**: "Advanced Agentic Coding & Macro Suite"

## Project Structure

The solution lives under `PowerX_Keys_V2_Rebuild/` with four primary projects:

```
PowerX_Keys_V2_Rebuild/
├── PowerX.Core/           ← Shared data models + constants (AppConfig, MacroItem, VersionInfo)
├── PowerX.Services/       ← Business logic
│   ├── Services/          ← ScriptCompilerService, ConfigManager, SupabaseAuthService, etc.
│   └── Managers/          ← ScriptManager, MacroDatabase, MacroTransferManager, TemplateDatabase
├── PowerX.UI/             ← Presentation layer
│   ├── ViewModels/        ← MVVM ViewModels (MainViewModel, MacroEditorViewModel, etc.)
│   ├── Views/             ← XAML views + code-behind
│   ├── Converters/        ← WPF value converters
│   ├── Helpers/           ← UI helpers
│   └── TrayIconManager.cs ← System tray
├── PowerX_Keys_V2/        ← Main app entry (App.xaml, MainWindow.xaml, project file)
│   ├── CoreData/          ← AutoHotkey64.exe binary
│   ├── Scripts/           ← Built-in AHK scripts (FindText.ahk, etc.)
│   ├── Resources/         ← Audio (pop.wav, tick.wav)
│   └── Themes/            ← XAML style resources
└── PowerX_Updater/        ← Standalone update installer
```

## MVVM Architecture

PowerX Keys follows the **Model-View-ViewModel (MVVM)** pattern:

### Models (in `PowerX.Core/Models/`)
- [AppConfig.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Core/Models/AppConfig.cs) — Master config (settings, hotkeys, profiles)
- [MacroItem.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Core/Models/MacroItem.cs) — Macro data model with steps hierarchy
- [ActionModels.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Core/Models/ActionModels.cs) — Action/hotkey binding models
- [VersionInfo.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Core/Models/VersionInfo.cs) — Static version constants

### ViewModels (in `PowerX.UI/ViewModels/`)
- [MainViewModel.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/ViewModels/MainViewModel.cs) — Root VM, navigation, profiles, engine state
- `MacroEditorViewModel` — Split across partial classes (Core, Commands, Properties, Recording, Capture, Optimization, SmartView, DragDrop)
- [ScriptLibraryViewModel.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/ViewModels/ScriptLibraryViewModel.cs) — Dashboard/library view
- [SettingsDashboardViewModel.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/ViewModels/SettingsDashboardViewModel.cs) — Settings management
- [AIAssistantViewModel.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/ViewModels/AIAssistantViewModel.cs) — AI chat integration
- [ViewModelBase.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/ViewModels/ViewModelBase.cs) — INotifyPropertyChanged base
- [RelayCommand.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/ViewModels/RelayCommand.cs) — ICommand implementation

### Views (in `PowerX.UI/Views/`)
- [MainWindow.xaml](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/MainWindow.xaml) — Shell window with sidebar + content area
- `MacroEditorView` — Full macro builder with drag-drop
- `ScriptLibraryView` — Dashboard showing all action cards
- `SettingsDashboardView` — Settings panel
- Specialty views: `ImageStudioWindow`, `CaptureOverlay`, `KeyCaptureWindow`, `WindowPickerWindow`, `AuthWindow`, `SubscriptionExpiredWindow`

### Services (in `PowerX.Services/Services/`)
- [ScriptCompilerService.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Services/Services/ScriptCompilerService.cs) — JSON config → AHK script compiler
- [MacroExecutionService.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Services/Services/MacroExecutionService.cs) — C# P/Invoke macro runner
- [ConfigManager.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Services/Services/ConfigManager.cs) — JSON config persistence
- [RemoteServerService.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Services/Services/RemoteServerService.cs) — Embedded HTTP server
- [HotKeyService.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Services/Services/HotKeyService.cs) — Windows hotkey registration
- [SupabaseAuthService.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Services/Services/SupabaseAuthService.cs) — Auth (email OTP / Google OAuth) + subscription checks

### Managers (in `PowerX.Services/Managers/`)
- [ScriptManager.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Services/Managers/ScriptManager.cs) — AHK process lifecycle (start/stop/reload)
- [MacroDatabase.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Services/Managers/MacroDatabase.cs) — SQLite macro storage
- [MacroTransferManager.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Services/Managers/MacroTransferManager.cs) — Import/export
- `TemplateDatabase` — Template storage
- ⚠️ `AILoginManager` (old OpenRouter PKCE login) has been **removed** — see [[ai-login-manager]]

### Converters (in `PowerX.UI/Converters/`)
- WPF value converters for data binding transformations
- Examples: `HotkeyToReadableConverter`, `LogicSummaryConverter`, `ProfileIconConverter`

## Key Architectural Insights

### 🎭 White-Labeled AHK Engine
`PowerX_Engine.exe` is a **copy of AutoHotkey64.exe** renamed to hide in Task Manager. Created dynamically by `ScriptManager.GetAhkExecutable()` if not present.

### ⚡ Dual Execution Model
- **AHK path** (primary): `ScriptCompilerService` generates `.ahk` scripts → `PowerX_Engine.exe` runs them
- **C# path** (secondary): `MacroExecutionService` uses Win32 P/Invoke (`mouse_event`, `keybd_event`, `SendKeys`)
- See [[dual-execution-model]] for details

### 🔒 Single Instance Enforcement
`App.xaml.cs` uses a named `Mutex` (`PowerX_Keys_V2_SingleInstanceMutex`) + Windows message broadcasting to restore the existing window instead of launching duplicates.

### 🛡️ Security-First Remote Server
The remote server is **always forced OFF on startup** regardless of saved config — a deliberate security measure.

### 🎯 Multi-Profile Engine
The script compiler generates a single `master_script.ahk` that contains hotkeys for **all running profiles simultaneously**. Profiles are additive — multiple can run at once.

### 📊 Performance Tracking via IPC
Stats flow from AHK → temp file → C# app. The AHK script maintains a `PendingExecutions` counter, flushed every 5 minutes or on-demand via `RegisterWindowMessage` IPC.

### 🚀 Turbo Engine Mode
When enabled (default OFF), the compiler injects a smart priority boost into the generated AHK script: each executed macro step calls `ProcessSetPriority("High")` and resets a rolling 3-second decay timer; once steps stop, the engine relaxes back to `Normal`. No C#-side priority management — the engine self-manages. Never uses `Realtime`.

## Startup Flow

1. `App.OnStartup()` → Mutex check → `ConfigManager.Initialize()` → `MacroDatabase.Initialize()`
2. Force remote server OFF → Apply Performance Mode if set
3. `SupabaseAuthService.InitializeAsync()` → restore saved session (or show `AuthWindow` login) → subscription check (`GetSubscriptionStatusAsync` → `IsSubscriptionValid`); invalid subscriptions open `SubscriptionExpiredWindow`
4. Create `MainWindow` → Ensure HWND (for hotkey service) → Show or start silent
5. `MainViewModel` constructor → Load profiles → Set default view → Auto-start engine if configured

## Key Files

- [PowerX_Keys_V2.csproj](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/PowerX_Keys_V2.csproj) — Project definition
- [App.xaml.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/App.xaml.cs) — Application entry point
- [MainWindow.xaml.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/MainWindow.xaml.cs) — Shell window
- [VersionInfo.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Core/Models/VersionInfo.cs) — Version constants

## Related Pages

- [[execution-pipeline]] — How macros compile from JSON → AHK
- [[dual-execution-model]] — AHK vs C# execution paths
- [[component-relationships]] — How all parts connect
