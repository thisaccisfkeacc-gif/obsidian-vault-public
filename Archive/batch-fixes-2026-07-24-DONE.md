# Log Archive — 2026-07-24 Batch Fixes

### Completed & Fixed Items (2026-07-24 Batch)
- **`Loop()` AHK Syntax Error**: Corrected `Loop({count})` to statement format `Loop {count}` across compiler (`ScriptCompilerService.cs:2039, 2213, 2514, 4919`).
- **WaitForKey C# Parameters**: Added support for `WaitKeyMode` (Strict OK / Strict Cancel) and modal styling in `MacroExecutionService.cs:1268`.
- **SetVariable AHK Escaping**: Added `` ` `` and `%` escaping to compiled AHK variable values in `ScriptCompilerService.cs:3103`.
- **CallMacro Call Stack Alignment**: Standardized `macroCallStack` in `MacroExecutionService.cs` to match macro tracking types and prevent recursive infinite loops.
- **UserInput Compiled `LastActionSucceeded`**: Set `LastActionSucceeded := 1` across YesNo and Dropdown GUI selection paths in `ScriptCompilerService.cs:3075`.
- **FileLauncher Compiled URL Detection**: Added `http://` / `https://` prefix check to `ScriptCompilerService.cs:3118`.
- **UI Element Block**: Unified 3-tier proximity matching across search loops (`MacroExecutionService.cs:2342`) and cleared auto-filled text when capturing input fields (`MacroEditorViewModel.Commands.cs:895`).
- **Pixel Search Block**: Removed redundant `PixelSearch(..., 0)` fallback call in preview (`SingleStep.cs:1005`).
- **Window Block**: Added system shell security blocklist (`cmd.exe`, `powershell.exe`, etc.) to compiled AHK scripts (`ScriptCompilerService.cs:2924`).
- **Logic IF Block**: Fixed WPF binding in `LogicContainerTemplates.xaml:110` from `LogicValue` to `LogicExpectedValue`.
- **WaitUntil Block**: Synchronized `timeoutMs` calculation (`MacroExecutionService.cs:3197`) and fixed `WindowActive` foreground check (`:3207`).
- **Mouse & Trace Block**: Fixed Right-Drag steps in C# (`MacroExecutionService.cs:1816`) and compiled AHK (`ScriptCompilerService.cs:2668`).
- **Variable Sanitization**: Preserved underscores (`_`) across step success states in `MacroExecutionService.cs:2315,2440`.
