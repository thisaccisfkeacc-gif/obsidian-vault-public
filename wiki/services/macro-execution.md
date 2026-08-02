---
tags: [service, macro, execution, pinvoke, windows-api]
date: 2026-05-23
sources:
  - Services/MacroExecutionService.cs
status: complete
---

# Macro Execution Service

C#-side macro playback engine using `user32.dll` P/Invoke. This is the **secondary execution path** used for live preview and debug mode — the primary path is the compiled AHK script via [[script-compiler]].

## Purpose

- Executes macros directly from the WPF app without the AHK engine
- Provides the "Test / Preview" functionality in the macro editor
- Supports all macro step types including logic branches, loops, and humanization
- Manages named targets and dynamic coordinate resolution

## Win32 API Imports

| Function | Purpose |
|----------|---------|
| `SetCursorPos` | Move mouse to absolute screen position |
| `GetCursorPos` | Read current mouse position |
| `mouse_event` | Simulate mouse clicks, scroll, drag (uses `UIntPtr` for `dwExtraInfo` compatibility) |
| `keybd_event` | Simulate keyboard input (uses `UIntPtr` for `dwExtraInfo` compatibility) |
| `MapVirtualKey` | Maps Virtual Key codes to hardware scan codes for DirectInput/game engine support |
| `VkKeyScan` | Convert character to virtual key code |
| `SetForegroundWindow` | Activate a window |
| `ShowWindow` | Minimize/maximize/restore windows |
| `FindWindow` | Find window by class/title |
| `MoveWindow` | Reposition/resize a window |
| `PostMessage` | Send async messages (e.g., `WM_CLOSE`) |
| `GetForegroundWindow` | Retrieve the active foreground window handle |
| `GetWindowRect` | Retrieve the bounding rectangle of a window |
| `GetWindowThreadProcessId` | Retrieve the process ID of a window handle |

## Virtual Key Code Map

The `GetVirtualKeyCode()` method maps 30+ key names to VK codes:
- Modifiers: `LShift` (0xA0), `RShift` (0xA1), `Ctrl` (0x11), `Alt` (0x12), `LWin` (0x5B)
- Navigation: `Up/Down/Left/Right`, `Home/End`, `PgUp/PgDn`
- Function keys: `F1`–`F24` (0x70–0x87)
- Special: `Enter` (0x0D), `Space` (0x20), `Tab` (0x09), `Escape` (0x1B)

## Execution Flow

1. **Minimize App** - hides the main window before playback
2. **Reset State** - clears `_lastActionSucceeded`, `_namedTargets`, `_stepSuccessStates`, `_userVariables`, and mouse button tracking flags (`_isLeftButtonDown`, `_isRightButtonDown`)
3. **Process Steps Recursively** - `ProcessStepCollectionAsync()` handles nesting
4. **Restore App** - brings the window back after execution, followed by releasing any held modifier keys and any mouse buttons that were left in a down state (conditional release prevents spurious desktop context menus)

## Step Processing

| Step Type | Behavior |
|-----------|----------|
| `MouseClick` | Moves cursor, fires click events. Supports: left/right/middle/double/multiple/hold/release/drag-and-drop, scroll, timer clicks, Mouse 4/5 (XButton1/2), visible-move interpolation (25 frames), and **Active Window targeting (clicking active window center)**. |
| `Keyboard` | `keybd_event()` with modifier combinations (e.g. `Ctrl+C`) parsing and sequential simulation, or low-level virtual key simulation for single special keys. Supports Hold Down / Released Up / Press. |
| `Delay` | `Task.Delay()` with humanization variance |
| `MouseTrace` | Delegates to [[smooth-trace-engine]] |
| `ImageSearch` / `PixelSearch` | Screen capture + pixel comparison (C# bitmap search) |
| `LogicIf` | Branch on: AboveStepSuccess, AboveStepFailed, NamedBlockSuccess, NamedBlockFailed, VariableEquals, VariableNotEquals. Skips evaluation entirely if the referenced source block is disabled or deleted. |
| `LoopSequence` | Repeats child steps N times |
| `GroupHeader` | Folder — recursively processes children |
| `UIElement` | Uses `System.Windows.Automation` to find/click/read elements. Supports Click, Read Text (stores to cleaned variable name), Wait Until Found (polling up to 10s), Check Exists, `ahk_exe` process-based window matching, and **Match Strategy (Exact, Find Latest, Find First)**. |
| `CallMacro` | Recursively executes target sub-macro in-process. Includes recursion and depth limit (10) check. |
| `Text` | Sends text using SendKeys. Supports variable resolution from `_userVariables`, fast paste mode (via Clipboard backup/restore), and proper escaping for closing brace `}` characters to prevent SendKeys crashes. |
| `UserInput` | Shows real input dialog (Yes/No prompt, Dropdown prompt with `SelectedText` helper, or Input prompt), supporting custom button labels for Yes/No options. Stores results in `_userVariables` and cleanly throws `OperationCanceledException` to abort the macro if cancelled/closed by the user. |
| `WaitUntil` | Loops up to `timeoutMs` checking `ImageFound` or `PixelFound` (via temporary single-step search AHK subprocesses) or **`WindowActive` (using foreground window matching)**. **ImageFound** tolerance uses the step's specific `ImageTolerance` property (instead of pixel tolerance). The test search script is compiled exactly once before the polling loop starts (reducing disk and process thrashing), with temporary file deletion guaranteed via a `finally` block. |
| `WaitForKey` | Pauses execution and displays a custom message box prompt representing continues/cancels okKey and cancelKey configurations. |
| `FileLauncher` | Launches the target file path, app, or URL using process execution. |
| `WindowAction` | Simulates window actions (Activate, Minimize, Maximize, Close, Restore). Supports **SmartWait polling (waiting up to 10s for slow windows in preview mode)**, **AutoLaunchIfMissing** (auto-launching the process if the window is missing), and optional active browser tab switching using UI Automation. |

## Keyboard Simulation Helpers

- **`SendKeyEvent(byte vk, string keyName, bool isDown)`** — Wrapper around `keybd_event`. Resolves the hardware scan code using `MapVirtualKey` and automatically applies the `KEYEVENTF_EXTENDEDKEY` flag if the key is detected as an extended key via `IsExtendedKey(vk)` or matches the `"numpadenter"` string.
- **`IsExtendedKey(byte vk)`** — Detects standard arrow keys, delete/insert/home/end/pgup/pgdn, right-modifier keys (`RCONTROL`, `RALT`), and numpad divide (`/`) to ensure appropriate extended key flags are sent.

## Humanization Engine

When `IsHumanized` is enabled:
- **Delay variance**: ±5% to ±60% based on level (1–4)
- **Mouse jitter**: ±2px random offset on click coordinates
- **Level 0**: uses macro's default humanization level

## Smart Target Resolution

Click steps resolve their target in priority order:
1. **Named target** — from a previous `ImageSearch` step (`_namedTargets` dictionary)
2. **Found target** — `_lastFoundX/Y` from most recent search
3. **Active Window target** — center coordinates of foreground window via `GetForegroundWindow` and `GetWindowRect`
4. **Fixed coordinates** — `step.X/Y` values
5. **Current position** — fallback to `GetCursorPos()`
6. **Dynamic offset** — shows `OffsetCaptureWindow` for user calibration
7. **Window Matching** — Uses `EnumWindows` to match window titles by substring (SetTitleMatchMode 2 equivalent) or process name if `ahk_exe` is used.

## UI Automation Integration

The `UIElement` step type uses `System.Windows.Automation`:
- Finds elements using combined multi-property matching (`AutomationId` AND `Name` AND `ClassName`) to avoid false positives.
- Window scoping via `SetTitleMatchMode 2` substring match or `ahk_exe` process name matching.
- Supports configurable timeouts and "Wait Until Found" / "Wait Until Gone" polling loops.
- Programmatic Background Mode:
  - If background mode is enabled, attempts to perform actions programmatically using UI Automation patterns (`InvokePattern` for click, `ValuePattern` for set text, `TogglePattern` for toggles, and `SetFocus` for focus).
  - If a pattern is not supported, it falls back to native Win32 `PostMessage` sending `WM_LBUTTONDOWN`/`WM_LBUTTONUP` (clicks), `WM_RBUTTONDOWN`/`WM_RBUTTONUP` (right clicks), or `WM_CHAR` character streams (text input) directly to the target window relative to the element's center coordinates. This prevents the physical mouse pointer from moving and does not force the window to the foreground.
  - If background mode is disabled, moves the physical mouse cursor to the element center and sends mouse/keyboard events.

## Key Files

- [MacroExecutionService.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Services/MacroExecutionService.cs)

## Related Pages

- [[script-compiler]]
- [[smooth-trace-engine]]
- [[macro-recording]]
- [[ui-element-capture]]
- [[find-text]]

