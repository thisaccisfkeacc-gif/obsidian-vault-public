---
tags:
  - architecture
  - execution
  - pinvoke
  - ahk
date: 2026-05-23
sources:
  - Services/MacroExecutionService.cs
  - Services/ScriptCompilerService.cs
  - Managers/ScriptManager.cs
status: complete
---

# Dual Execution Model

PowerX Keys has **two completely separate macro execution engines**. This page explains when each is used, why both exist, and how they differ.

## The Two Engines

### 🔵 AHK Engine (Primary — Production)

- **Service**: [ScriptCompilerService.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Services/ScriptCompilerService.cs) + [ScriptManager.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Managers/ScriptManager.cs)
- **Runtime**: `PowerX_Engine.exe` (executing `master_script.ahk` for hotkey listening and `executor_script.ahk` for macro execution)
- **Trigger**: User clicks **START** on the dashboard
- **How it works**: Hotkey listener process (`master_script.ahk`) captures triggers and communicates with the execution process (`executor_script.ahk`) via Win32 messages (0x5555).
- **Runs in**: Separate OS processes (invisible, no tray icons)

### 🟣 C# Engine (Secondary — Preview)

- **Service**: [MacroExecutionService.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Services/MacroExecutionService.cs)
- **Runtime**: In-process via Win32 P/Invoke calls
- **Trigger**: User clicks **PREVIEW** in the Macro Editor
- **How it works**: Direct C# code calling `mouse_event`, `keybd_event`, `SendKeys`
- **Runs in**: Same process as the WPF app (async on UI thread)

## When Each Engine Is Used

| Scenario | Engine | Why |
|---|---|---|
| **Dashboard START** | AHK 🔵 | Full-speed, OS-level hotkey interception |
| **Macro Editor PREVIEW** | C# 🟣 | No compilation needed, instant feedback |
| **Schedule triggers** | AHK 🔵 | Must run even when app is minimized |
| **Screen event detection** | AHK 🔵 | Polling `ImageSearch`/`PixelSearch` in background |
| **Mobile remote triggers** | Neither* | Handled by HTTP server → then AHK or C# |
| **Test/sandbox execution** | AHK 🔵 | Uses dedicated `test_runner.ahk` slot |
| **Single-step testing** (Image/Pixel search in preview) | AHK 🔵 | Compiles a temp test script for real search results |

## Why Both Engines Exist

### AHK Engine Strengths 💪
- **OS-level hotkey hooking** — Intercepts keys before any application
- **Background execution** — Runs independently of the GUI process
- **Native AHK features** — `ImageSearch`, `PixelSearch`, `WinActive` scoping, `#HotIf` directives
- **Performance** — Dedicated process with optional High priority
- **Resilience** — Survives even if the main app crashes

### C# Engine Strengths 💪
- **Instant preview** — No compilation/process launch overhead
- **Debug integration** — Can highlight steps, show search results, show prompts
- **UI interaction** — Can show WPF dialogs mid-execution (popups, input prompts)
- **Dynamic offset capture** — Interactive `OffsetCaptureWindow` for click positioning
- **Humanization** — Built-in jitter and timing randomization for natural-looking input

## C# Engine (MacroExecutionService) Deep Dive

### Win32 P/Invoke APIs Used

```csharp
// Mouse control
[DllImport("user32.dll")] SetCursorPos(int x, int y)
[DllImport("user32.dll")] GetCursorPos(ref Win32Point pt)
[DllImport("user32.dll")] mouse_event(int flags, int dx, int dy, int data, int extra)

// Keyboard control
[DllImport("user32.dll")] keybd_event(byte vk, byte scan, int flags, int extra)
[DllImport("user32.dll")] VkKeyScan(char ch)

// Window management
[DllImport("user32.dll")] SetForegroundWindow(IntPtr hWnd)
[DllImport("user32.dll")] ShowWindow(IntPtr hWnd, int cmd)
[DllImport("user32.dll")] FindWindow(string className, string windowName)
[DllImport("user32.dll")] MoveWindow(IntPtr hWnd, int x, int y, int w, int h, bool repaint)
[DllImport("user32.dll")] PostMessage(IntPtr hWnd, uint msg, IntPtr wParam, IntPtr lParam)
```

### Execution Flow

```
ExecuteMacroAsync(macro)
    ├── Minimize main window
    ├── ProcessStepCollectionAsync(steps) ← recursive
    │     ├── Skip disabled steps
    │     ├── Handle LogicIf/Else branches
    │     ├── Handle GroupHeader (folders)
    │     ├── Handle LoopSequence (repeats)
    │     └── ExecuteStepAsync(step) per step type:
    │           ├── MouseClick → SetCursorPos + mouse_event
    │           ├── Delay → Task.Delay
    │           ├── Keyboard → keybd_event or SendKeys.SendWait
    │           ├── Text → char-by-char SendKeys with delays
    │           ├── Popup → DarkMessageBoxWindow.Show
    │           ├── ImageSearch/PixelSearch → compile temp AHK test script!
    │           ├── MouseTrace → SmoothTraceEngine playback
    │           ├── WindowAction → FindWindow + ShowWindow/MoveWindow
    │           ├── UIElement → UI Automation API
    │           ├── CallMacro → recursive ProcessStepCollectionAsync call
    │           └── SystemSound → SystemSounds.Play
    └── Restore main window
```

### Notable C# Engine Features

- **Cancellation support** — Every step checks `CancellationToken`
- **Step success tracking** — `_lastActionSucceeded` + `_stepSuccessStates` dictionary for logic branching
- **Named targets** — `_namedTargets` maps step names to found coordinates for cross-step targeting
- **Visible mouse movement** — 25-frame interpolation for smooth cursor travel
- **Mouse return-to-origin** — Clicks at target then returns cursor to starting position
- **Dynamic offset** — Interactive capture window for precise click offset adjustment
- **Humanization engine** — 4 levels of timing randomization (5% to 60% variance)
- **Playback speed** — `multiplier` parameter scales all delays
- **Drag and drop** — Full support including right-drag with visible movement

### Hybrid Approach for Image/Pixel Search

Even in **C# preview mode**, `ImageSearch` and `PixelSearch` steps **delegate to AHK**! 🤯

```csharp
// MacroExecutionService line 621
string testScriptPath = ScriptCompilerService.CompileSingleStepTestScript(step);
// ... launches AHK with the test script, captures stdout for coordinates
```

This is because AHK's `ImageSearch` and `PixelSearch` are highly optimized native functions — C# would need third-party libraries to replicate them. The test script outputs `FOUND:x,y` to stdout.

## Comparison Table

| Feature | AHK Engine 🔵 | C# Engine 🟣 |
|---|---|---|
| **Process** | Separate (`PowerX_Engine.exe`) | In-app (async) |
| **Startup time** | ~1.5s (compile + launch + AV delay) | Instant |
| **Hotkey hooking** | ✅ OS-level | ❌ Not available |
| **Background operation** | ✅ Independent process | ❌ Needs app running |
| **UI dialogs** | AHK native (`MsgBox`, `InputBox`) | WPF dialogs |
| **Image/Pixel search** | ✅ Native | Delegates to AHK |
| **Debug highlighting** | ❌ | ✅ Visual step feedback |
| **Humanization** | ❌ (would need AHK scripting) | ✅ Built-in jitter |
| **Mouse physics** | ❌ | ✅ Smooth trace engine |
| **Cancellation** | Kill process | CancellationToken |
| **Use case** | Production execution | Development preview |

## Key Files

- [MacroExecutionService.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Services/MacroExecutionService.cs) — C# execution engine (882 lines)
- [ScriptCompilerService.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Services/ScriptCompilerService.cs) — AHK script generator (2,227 lines)
- [ScriptCompilerService.SingleStep.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Services/ScriptCompilerService.SingleStep.cs) — Single-step test script compiler
- [ScriptManager.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Managers/ScriptManager.cs) — AHK process manager
- [SmoothTraceEngine.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Services/SmoothTraceEngine.cs) — Mouse trace playback (C# only)

## Related Pages

- [[overview]] — Full architecture overview
- [[execution-pipeline]] — Detailed AHK compilation pipeline
- [[component-relationships]] — How engines fit into the system
