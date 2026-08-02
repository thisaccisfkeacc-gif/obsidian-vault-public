---
tags:
  - architecture
  - components
  - data-flow
  - diagram
date: 2026-05-23
last_updated: 2026-08-01
sources:
  - PowerX_Keys_V2/MainWindow.xaml.cs
  - PowerX.UI/ViewModels/MainViewModel.cs
  - PowerX.Services/Managers/ScriptManager.cs
  - PowerX.Services/Services/ScriptCompilerService.cs
status: complete
---

# Component Relationships

This page maps how the major components of PowerX Keys connect, interact, and flow data between each other.

## System Architecture Diagram

```mermaid
graph TB
    subgraph UI["UI Layer (WPF)"]
        MW["MainWindow"]
        SLV["ScriptLibraryView"]
        MEV["MacroEditorView"]
        SDV["SettingsDashboardView"]
        AAV["AIAssistantView"]
    end

    subgraph VM["ViewModel Layer"]
        MVM["MainViewModel"]
        SLVM["ScriptLibraryViewModel"]
        MEVM["MacroEditorViewModel"]
        SDVM["SettingsDashboardViewModel"]
        AAVM["AIAssistantViewModel"]
    end

    subgraph SVC["Service Layer"]
        CM["ConfigManager"]
        SCS["ScriptCompilerService"]
        MES["MacroExecutionService"]
        HKS["HotKeyService"]
        RSS["RemoteServerService"]
        AUS["AutoUpdateService"]
    end

    subgraph MGR["Manager Layer"]
        SM["ScriptManager"]
        MDB["MacroDatabase"]
        SAS["SupabaseAuthService"]
        TIM["TrayIconManager"]
    end

    subgraph EXT["External Processes"]
        PE["PowerX_Engine.exe"]
        AHK["master_script.ahk"]
    end

    MW -->|DataContext| MVM
    SLV -->|DataContext| SLVM
    MEV -->|DataContext| MEVM
    SDV -->|DataContext| SDVM
    AAV -->|DataContext| AAVM

    MVM -->|CurrentView| SLVM
    MVM -->|CurrentView| MEVM
    MVM -->|CurrentView| SDVM

    MVM -->|Start/Stop| SM
    MW -->|RunButton_Click| SM
    SM -->|Compiles| SCS
    SCS -->|Reads| CM
    SCS -->|Reads macros| MDB
    SCS -->|Writes| AHK
    SM -->|Launches| PE
    PE -->|Runs| AHK

    MEVM -->|Preview| MES
    MES -->|P/Invoke| PE

    CM -->|Persists JSON| CM
    MDB -->|SQLite| MDB

    HKS -->|Window handle| MW
    TIM -->|System tray| MW
    RSS -->|HTTP server| MVM
    SAS -->|Auth & subscription| MVM
```

## MainWindow → ViewModel Connection

[MainWindow](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/MainWindow.xaml.cs) is the **shell** — it creates and owns the `MainViewModel`:

```csharp
// MainWindow constructor (line 105-110)
var vm = new ViewModels.MainViewModel();
this.DataContext = vm;
```

Key responsibilities of MainWindow:
- **Run/Stop button** — `RunButton_Click()` delegates to `ScriptManager.Start()`/`Stop()` or `MacroExecutionService.ExecuteMacroAsync()` depending on context
- **Status banner** — `UpdateStatusBanner()` reflects engine state (START/RUNNING/PREVIEW)
- **HotKey registration** — `OnSourceInitialized()` initializes `HotKeyService` with the window handle
- **Window restore** — Listens for `WM_RESTORE_WINDOW` broadcast message via `HwndMessageHook`
- **Engine exit handler** — Subscribes to `ScriptManager.EngineExited` to update UI
- **Drag-drop profiles** — Handles sidebar drag-and-drop for moving macros between profiles

## MainViewModel — The Central Hub

[MainViewModel](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/ViewModels/MainViewModel.cs) is the **application coordinator**. It's a singleton (`Instance` property) that manages:

### Navigation
```
CurrentView property → swaps between:
├── ScriptLibraryViewModel (default home)
├── MacroEditorViewModel (macro builder)
├── SettingsDashboardViewModel (settings)
└── SettingsViewModel (legacy)
```

Navigation has **safety guards** — if the Macro Editor has unsaved changes (`IsDirty`), navigation is blocked and an unsaved warning is triggered.

### Profile Management
- `ActiveProfile` — Currently selected profile (default: "CustomActions")
- `RunningProfiles` — `ObservableCollection<string>` of profiles currently executing
- `Profiles` — All available profiles from `ConfigManager`
- Profile changes trigger `ScriptLibraryViewModel.RefreshLibrary()` and macro re-binding

### Engine State
- `IsEngineRunning` — Two-way sync with `ScriptManager.IsRunning`
- Setting to `true` → minimizes window → calls `ScriptManager.Start()` on background thread
- Setting to `false` → calls `ScriptManager.Stop()`, clears running profiles
- Listens to `ScriptManager.EngineExited` event for unexpected exits

### Auto-Start Logic
On construction, if `AutoStartEngineOnLaunch` is enabled:
1. Restores previously active profiles from saved state
2. Delays 3.5 seconds (for stable Windows boot)
3. Calls `ScriptManager.Start()`

## Data Flow Diagrams

### Starting the Engine

```mermaid
sequenceDiagram
    participant User
    participant MW as MainWindow
    participant MVM as MainViewModel
    participant SM as ScriptManager
    participant SCS as ScriptCompilerService
    participant CM as ConfigManager
    participant MDB as MacroDatabase
    participant PE as PowerX_Engine.exe

    User->>MW: Click START button
    MW->>MW: RunButton_Click()
    MW->>MVM: RunningProfiles.Add(activeProfile)
    MW->>SM: Start()
    SM->>SCS: CompileMasterScript()
    SCS->>CM: Read hotkeys + settings
    SCS->>MDB: LoadAllMacros()
    SCS->>SCS: Generate AHK script
    SCS-->>SM: master_script.ahk written
    SM->>SM: Sleep(200ms) for AV
    SM->>SM: GetAhkExecutable()
    SM->>PE: Process.Start("master_script.ahk")
    PE-->>SM: Process running (PID)
    SM->>SM: StartProcess(KEY_EXECUTOR) → executor_script.ahk
    SM-->>MW: return true
    MW->>MW: UpdateStatusBanner() → "RUNNING"
```

### Previewing a Macro

```mermaid
sequenceDiagram
    participant User
    participant MW as MainWindow
    participant MES as MacroExecutionService
    participant AHK as AHK Test Script

    User->>MW: Click PREVIEW button
    MW->>MW: RunButton_Click() [macro editor mode]
    MW->>MES: ExecuteMacroAsync(macro)
    MES->>MES: Minimize window
    MES->>MES: ProcessStepCollectionAsync()
    
    loop Each MacroStep
        alt MouseClick / Keyboard / Text
            MES->>MES: P/Invoke (mouse_event, keybd_event)
        else ImageSearch / PixelSearch
            MES->>AHK: CompileSingleStepTestScript()
            AHK-->>MES: stdout "FOUND:x,y"
        else Delay
            MES->>MES: Task.Delay()
        end
    end
    
    MES->>MES: Restore window
    MES-->>MW: Complete
    MW->>MW: UpdateStatusBanner() → "PREVIEW"
```

### Stats Flow (AHK → C#)

```mermaid
sequenceDiagram
    participant AHK as AHK Engine
    participant FS as File System
    participant CM as ConfigManager
    participant MVM as MainViewModel

    Note over AHK: Macro executes
    AHK->>AHK: PendingExecutions += 1

    alt Every 5 minutes
        AHK->>FS: FileAppend(count, PowerX_MacroStats.txt)
    else On-demand (user opens Analytics)
        CM->>AHK: PostMessage(PowerX_FlushStats)
        AHK->>FS: FileAppend(count, PowerX_MacroStats.txt)
    else On engine exit
        AHK->>FS: FlushStats()
    end

    CM->>FS: Read PowerX_MacroStats.txt
    CM->>CM: Update TotalMacrosExecuted
    CM->>MVM: RefreshStats()
```

## Service Dependencies

```mermaid
graph LR
    subgraph "Startup Order"
        A["ConfigManager.Initialize()"] --> B["MacroDatabase.Initialize()"]
        B --> C["MainWindow()"]
        C --> D["HotKeyService.Initialize(hwnd)"]
        C --> E["MainViewModel()"]
        E --> F["ScriptManager.Start() (if auto-start)"]
    end
```

### ConfigManager — The Config Hub

Everything reads from `ConfigManager.Current`:
- `ScriptCompilerService` reads hotkeys/settings to generate scripts
- `ScriptLibraryViewModel` reads hotkeys to display action cards
- `MacroEditorViewModel` reads/writes individual macro bindings
- `SettingsDashboardViewModel` reads/writes all settings
- `MainViewModel` reads profiles, engine settings, last active profile

### MacroDatabase — The Macro Store

- **Write path**: `MacroEditorViewModel` → `MacroDatabase.SaveMacroAsync()`
- **Read path**: `ScriptCompilerService` → `MacroDatabase.LoadAllMacros()` (during compilation)
- **Read path**: `MainViewModel` → `MacroDatabase.LoadAllMacrosAsync()` (on startup)
- **Delete path**: `MainViewModel.DeleteMacroCommand` → `MacroDatabase.DeleteMacroAsync()`

## Component Summary Table

| Component | Type | Role | Key Connections |
|---|---|---|---|
| `MainWindow` | View | Shell, routing, status | → `MainViewModel`, `ScriptManager`, `HotKeyService` |
| `MainViewModel` | ViewModel | Navigation, profiles, engine | → `ScriptManager`, `ConfigManager`, all child VMs |
| `ScriptLibraryViewModel` | ViewModel | Dashboard cards | → `ConfigManager`, `MacroDatabase` |
| `MacroEditorViewModel` | ViewModel | Macro building | → `MacroDatabase`, `MacroExecutionService` |
| `ScriptCompilerService` | Service | AHK code generation | → `ConfigManager`, `MacroDatabase` |
| `MacroExecutionService` | Service | C# macro execution | → Win32 API, `ScriptCompilerService` (for search) |
| `ScriptManager` | Manager | AHK process lifecycle | → `ScriptCompilerService`, `PowerX_Engine.exe` |
| `ConfigManager` | Service | JSON persistence | → File system |
| `MacroDatabase` | Manager | SQLite storage | → File system |
| `HotKeyService` | Service | Global hotkeys | → `MainWindow` (HWND) |
| `TrayIconManager` | Service | System tray | → `MainWindow` |
| `SupabaseAuthService` | Service | Auth (OTP/Google), subscription checks | → `AuthWindow`, `MainViewModel`, `SettingsDashboardViewModel` |
| `RemoteServerService` | Service | Mobile HTTP server | → `ConfigManager`, `ScriptManager` |

## Key Files

- [MainWindow.xaml.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/MainWindow.xaml.cs) — Shell window code-behind
- [MainViewModel.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/ViewModels/MainViewModel.cs) — Central application ViewModel
- [ScriptManager.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Services/Managers/ScriptManager.cs) — Engine process manager
- [ConfigManager.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Services/Services/ConfigManager.cs) — Configuration hub
- [MacroDatabase.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Services/Managers/MacroDatabase.cs) — SQLite macro storage
- [SupabaseAuthService.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Services/Services/SupabaseAuthService.cs) — Auth & subscription service

## Related Pages

- [[overview]] — Full architecture overview
- [[execution-pipeline]] — Detailed AHK compilation pipeline
- [[dual-execution-model]] — AHK vs C# execution comparison
