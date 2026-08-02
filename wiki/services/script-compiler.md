---
tags: [service, compiler, ahk, engine, core]
date: 2026-08-01
sources:
  - Services/ScriptCompilerService.cs
status: complete
---

# Script Compiler Service

The **brain** of PowerX Keys. `ScriptCompilerService` transforms the JSON-defined hotkey/macro configuration into a fully self-contained AutoHotkey v2 script that `PowerX_Engine.exe` executes in the background.

## Purpose

- Reads all enabled hotkeys/macros from [[config-manager]]
- Generates a single `master_script.ahk` file in `%DOCUMENTS%\PowerX_Keys\Engine\`
- Handles multi-profile compilation — only hotkeys and text snippets from **running profiles** (including `"TextSnippets"`) are compiled
- Supports 11 trigger modes (from `PowerX.Core\Models\AppEnums.cs`): Single, DoubleTap, Hold, Release, LongPress, Toggle, ScreenEvent, Schedule, MobileRemote, PressAndRelease, ScheduledTime
- Produces AHK code for 15+ macro step types

## Architecture

```mermaid
graph LR
    A["JSON Config"] --> B["ScriptCompilerService"]
    B --> C["master_script.ahk"]
    C --> D["PowerX_Engine.exe"]
    D --> E["Windows Input"]
```

## Compilation Pipeline

1. **Load Config** — reads `ConfigManager.Current` and all macros from [[database-schema]]
2. **Dependency Check** — scans all macros for `ImageSearch` steps with `UseFastEngine=true`; if found, copies `FindText.ahk` to the engine folder
3. **Script Header** — generates `#Requires AutoHotkey v2.0`, coordinate modes, `#SingleInstance Force`, `#NoTrayIcon`
4. **Kill Switch** — configurable panic button (default: Shift + Escape)
5. **Ghost Keyboard Reset** — `OnExit` handler releases stuck modifier keys
6. **Stats Tracker** — `PendingExecutions` counter with 5-minute auto-flush to `%TEMP%\PowerX_MacroStats.txt`
7. **IPC Bridge** — `RegisterWindowMessage("PowerX_FlushStats")` for C#→AHK stat sync
8. **Hotkey Compilation** — iterates each active hotkey and generates the appropriate AHK block (properly closed loop to avoid duplication)
9. **Global Snippets & Helpers** — appends global text snippets (if the `"TextSnippets"` profile is running) and the `GetHumanizedDelay` helper function exactly once at the end of the script, then writes the completed file to disk. Snippets play the custom sound `Resources/Notification_11.wav` using `DllCall("winmm.dll\PlaySoundW", ...)` inside a `try` block, bypassing AutoHotkey's 127-character file path limit on the native `SoundPlay` function.

## Trigger Modes

| Mode | AHK Output | Notes |
|------|-----------|-------|
| **Single** | `~key::` | Standard hotkey |
| **Hold** | `KeyWait` with timeout | Press-and-hold detection |
| **Schedule** | `SetTimer(func, interval)` | Named function + timer, no hotkey |
| **ScreenEvent** | `SetTimer` + `ImageSearch`/`PixelSearch` | Polling loop with configurable bounds |
| **MobileRemote** | *(skipped)* | Handled by [[remote-server]] via HTTP |

## Mouse Trigger Compilation 🖱️

When a mouse trigger (Middle Click, Side Buttons, modifier combos, or button-button combinations) is compiled, special translation rules apply:

- **Tilde Pass-through (`~`)**: Mouse triggers are automatically prefixed with `~` in the hotkey header to ensure the default click action is sent to the operating system, preventing mouse lockouts.
- **Brace Stripping**: Curly braces `{` and `}` are stripped from hotkey headers (e.g., `~^+LButton::` instead of `~^+{LButton}::`).
- **Strict Modifier Combinations**: AutoHotkey v2 does not natively support modifiers on custom combinations (e.g. `^XButton1 & XButton2`). The compiler formats the header as a clean combination `~XButton1 & XButton2::`, and injects strict modifier checks at the beginning of the hotkey body (e.g., `if (!GetKeyState("Ctrl", "P")) { return }`) to ensure it only triggers when modifiers are held down.

## Keyboard Combo Compilation ⌨️

When the **Advanced Trigger Combos** toggle is enabled, users can capture two non-modifier keys pressed simultaneously (e.g. `Tab & A`, `F1 & 1`, `~ & B`). The compiler handles these in `FormatHotkeyHeader`:

- **Detection**: A combo is identified by the presence of `&` in the hotkey string *without* any mouse button tokens (`LButton`, `MButton`, `XButton`, etc.).
- **Modifier Stripping**: Standard modifier prefixes (`^`, `!`, `+`, `#`) are stripped from the combo header — they don't apply to custom AHK combinations.
- **Tilde Pass-through (`~`)**: The header is prefixed with `~` so the first key's native action still reaches the OS (e.g. `~Tab & a::` lets Tab still indent).
- **AHK-friendly Key Names**: `KeyCaptureWindow` uses `FormatKeyName()` to convert WPF `Key` enum values into their AHK equivalents (e.g. `Oem3` → `` ` ``, `OemTilde` → `~`).

### Example

| User Capture | Raw Header | Compiled AHK |
|---|---|---|
| Tab + A | `Tab & A` | `~Tab & a::` |
| F1 + 1 | `F1 & 1` | `~F1 & 1::` |
| ~ + B | `~ & B` | `~~ & b::` |

## Macro Step Types Compiled

- `Delay` → `Sleep(ms)`
- `Text` → `SendEvent("{Text}" . "...")` or fast-paste via clipboard
- `Keyboard` → `Send("{Blind}" . "key")` with Hold Down / Released Up, brace-escaping for single special key presses, and combo double-wrapping resolution.
- `MouseClick` → `Click(x, y, "Button")` with scroll, move-only variants
- `Popup` → `MsgBox()`
- `Notification` → `TrayTip`
- `UserInput` → `InputBox()`, dropdown GUI, or yes/no custom GUI
- `WaitForKey` → Custom styled dark theme borderless tooltip (smoothly following the mouse cursor) or custom styled dark theme borderless GUI popup (with styled text-button controls and mouse click-drag support) depending on selected style (3 modes: Specific, StrictOK, StrictCancel)
- `FileLauncher` → `Run("path")`
- `SystemSound` → `SoundPlay("*-1")`
- `LoopSequence` → `Loop N { ... }`
- `ImageSearch` → AHK `ImageSearch` or [[find-text]] `FindText()` for fast engine
- `PixelSearch` → AHK `PixelSearch`
- `WaitUntil` → Loops up to `timeoutMs` checking AHK `ImageSearch`, `PixelSearch`, or `WinActive` active window status.
- `UIElement` → AHK COM-based UI Automation element lookup. Supports Click, Read Text, Check Exists, Wait Until Found, and Match Strategy (Exact, Find First, Find Latest).
- `WindowAction` → Activates, closes, minimizes, maximizes, or restores a window. Supports smart wait, position/size memory, auto-launching, and optional COM-based active browser tab switching.


## Toggle Slot Compilation

- **Shared Pipeline** — Toggle and Schedule slot macros are compiled using the unified, robust `CompileStepCollection` local function by passing `skipTokenCheck: true`.
- **Advanced Features** — Unifying the compiler methods automatically adds full support for all 21 step types (including Text, Mouse Click, launchers, loops, and logic branches) in Toggle and Schedule modes, resolving the previous limitation where only Keyboard and Delay steps compiled.
- **Global State Initialization** — The compiler declares the global toggle variable `global Toggle_xxxx := 0` outside the reset function block (in the global namespace scope) to prevent AHK v2 runtime variable initialization warning/error crashes.
- **Compiler State Isolation** — The compiler guarantees state isolation by saving and restoring `targetMacro` to the current `slotMacro` scope, and restoring the core compiler StringBuilder (`sb = loopOrigSb`) before any `continue` statements inside the hotkey compilation loop, preventing master/executor script cross-corruption.

## Scoping Rules

Each hotkey can have app-scoping:
- **Global** — `#HotIf` (no condition)
- **Include** — `GroupAdd` + `#HotIf WinActive("ahk_group ...")`
- **Exclude** — `GroupAdd` + `#HotIf !WinActive("ahk_group ...")`

## Smart Window Matching (WIN_LIVE Scopes)

When a macro step uses `WIN_LIVE` scope (captured via window picker), the compiler generates a **3-step matching hierarchy** for robust window targeting:

| Step | Method | AHK Code | When Used |
|------|--------|----------|-----------|
| 1 | **Title + Process** | `WinExist("title ahk_exe proc")` | Exact window (e.g., specific browser tab) |
| 2 | **Most Recent Process Window** | `WinGetList("ahk_exe proc")` → first handle | Title changed (e.g., navigated to different page) |
| 3 | **Fallback Coordinates** | Static `x1,y1,x2,y2` | No matching window found at all |

- Title is captured during window selection via `GetWindowText` P/Invoke in `CaptureOverlay.xaml.cs`
- Format: `WIN_LIVE:processName:TITLE=windowTitle:x1,y1,x2,y2`
- **Colon-safe parsing**: Titles containing colons (e.g., `localhost:3000 - Chrome`) are handled by finding the **last colon followed by a digit** instead of the first colon
- `SetTitleMatchMode 2` enables substring matching for flexible title targeting
- Backward compatible — old `WIN_LIVE` strings without `TITLE=` skip directly to step 2

## Variable System

- **Global Variable Mapping** — All user variables are stored in a global AHK Map: `global UserVariables := Map("UserText", "")`.
- **Global Scope** — Variables are shared globally across all hotkeys and macros.
- **Reference Resolution** — `Text` steps reference variables via `(UserVariables.Has("varRef") ? UserVariables["varRef"] : "")` in all compilation pipelines, preventing undeclared/unassigned variable compiler errors.
- **Success Tracking** — Named block execution states are recorded in `global StepSuccessStates := Map()`. Logic If branching reads from this map (e.g. `StepSuccessStates["MyStep"] == 1`), eliminating load-time validation failures when referenced steps are disabled or deleted.
- **Humanization Level Fallback** — When `step.HumanizationLevel == 0` (Macro Default) and `step.IsHumanized` is true, the compiled AHK script resolves the level parameter to `targetMacro.DefaultHumanizationLevel` to ensure the default humanization level is applied.

## Key Methods

| Method | Description |
|--------|-------------|
| `CompileMasterScript()` | Main entry point — generates the entire script |
| `CompileScheduleSteps()` | Recursive local function for scheduled macro steps |

## Key Files

- [ScriptCompilerService.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Services/Services/ScriptCompilerService.cs)
- [FindText.ahk](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Scripts/FindText.ahk) — dependency copied for fast engine

## Related Pages


- [[config-manager]]
- [[macro-execution]]
- [[find-text]]
- [[smooth-trace-engine]]
- [[macro-recording]]
