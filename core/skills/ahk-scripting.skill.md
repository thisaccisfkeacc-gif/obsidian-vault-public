---
name: ahk-scripting
description: AutoHotkey v2 scripting knowledge for PowerX Keys. Use when working with ScriptCompilerService.cs (AHK code generation), debugging generated .ahk files, understanding hotkey binding syntax, or adding new macro step types that require AHK output.
tags: [skill, ahk, autohotkey, scripting, compiler]
date: 2026-06-08
status: active
sources:
  - wiki/services/script-compiler.md
  - wiki/architecture/execution-pipeline.md
  - CoreData/engine scripts
---

# 🔧 Skill: AutoHotkey v2 Scripting

PowerX Keys uses AutoHotkey v2 (not v1). The AHK script is **always auto-generated** by `ScriptCompilerService.cs`. Never edit generated scripts directly.

## The Generation Pipeline

```
MacroItem (SQLite) + AppConfig (JSON)
         ↓
ScriptCompilerService.CompileMasterScript()
         ↓
.ahk file in %DOCUMENTS%/PowerX_Keys/Engine/
         ↓
PowerX_Engine.exe (= AutoHotkey64.exe renamed) runs it
```

## Key AHK v2 Syntax Reference

### Hotkey Bindings
```autohotkey
; Basic hotkey
^!a::  ; Ctrl+Alt+A
{
    ; code here
}

; Mouse button hotkeys
XButton1::  ; Mouse button 4
XButton2::  ; Mouse button 5
MButton::   ; Middle click

; With modifiers
!^+a::  ; Alt+Ctrl+Shift+A (order: !^#+)
```

**Modifier symbols:**
- `!` = Alt
- `^` = Ctrl
- `+` = Shift
- `#` = Win
- `*` = Any modifier (wildcard)

### Key Send Operations
```autohotkey
; Send text
Send "Hello World"
SendInput "faster text"  ; Preferred — bypasses hooks

; Send key combos
Send "^c"        ; Ctrl+C
Send "{Enter}"   ; Enter key
Send "{F1}"      ; Function key

; For macro steps, use SendInput (faster, safer):
SendInput "{" . keyName . "}"
```

### Mouse Operations
```autohotkey
; Click at coordinates
Click 500, 300
Click "right", 500, 300  ; Right click

; Move without clicking
MouseMove 500, 300, 5  ; speed=5

; For precise macro playback (NOT AHK — use SmoothTraceEngine.cs instead)
```

### Sleep / Delay
```autohotkey
Sleep 100   ; 100ms
Sleep 1000  ; 1 second
```

### Image Search (FindText integration)
```autohotkey
; PowerX Keys uses Feiyue's FindText library
; Generated code looks like:
FindText := A_ScriptDir . "\FindText.ahk"
#Include %FindText%

result := FindText(x1, y1, x2, y2, "imageData")
```

### Loops and Conditions
```autohotkey
; Loop
Loop 5 {
    Send "{Space}"
    Sleep 50
}

; If condition
if (result = "success") {
    Send "done"
}
```

## ScriptCompilerService.cs — How to Add a New Step Type

1. **Find the switch-case** in `CompileMasterScript()` (or `GenerateStepCode()`)
2. **Add a new case** for your `MacroStepType` enum value
3. **Return valid AHK v2 code** as a string
4. **Test** by running a macro with your new step and checking the generated `.ahk` file

Example pattern (C# side):
```csharp
case MacroStepType.YourNewStep:
    return $"Send \"{step.Value}\"{Environment.NewLine}";
```

## Debugging Generated Scripts

1. Find the generated script: `%DOCUMENTS%/PowerX_Keys/Engine/master.ahk`
2. Look for syntax errors — AHK v2 is strict about:
   - String quotes: use `"` not `'`
   - No implicit concatenation — use `.` operator
   - Functions use `()` — not `%var%` style (that's AHK v1)
3. Run the `.ahk` directly with `PowerX_Engine.exe` for isolated testing

## Common AHK v2 Gotchas (vs v1)

| AHK v1 | AHK v2 | Notes |
|--------|--------|-------|
| `MsgBox, text` | `MsgBox "text"` | v2 uses parens/quotes |
| `%var%` | `var` | No percent signs for variables |
| `StringLen, out, in` | `StrLen(str)` | v2 uses functions |
| `Loop, 5` | `Loop 5` | No comma |
| `#NoEnv` | Not needed | v2 always has this |
| `SetTitleMatchMode, 2` | `SetTitleMatchMode 2` | No comma |

## AHK and Thread Safety

- AHK is single-threaded per script
- `ScriptManager` handles 4 process slots (master, macro, tester, recorder)
- Never assume AHK shares state with C# — they communicate only via:
  - Process start/stop (`ScriptManager.cs`)
  - File system (generated scripts, trace CSV files)
  - No IPC — they are isolated processes

## Related Files

- [ScriptCompilerService.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Services/ScriptCompilerService.cs) — AHK code generator (163KB, largest file)
- [ScriptManager.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Managers/ScriptManager.cs) — Process lifecycle
- [[script-compiler]] — Wiki page with full overview
- [[execution-pipeline]] — How everything connects
