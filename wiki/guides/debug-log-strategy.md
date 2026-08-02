---
title: Debug Log Strategy
category: guides
created: 2026-06-16
---

# 🔍 Debug Log Strategy — The Nuclear Debugging Tool

> [!IMPORTANT]
> **When you've tried 2+ theory-based fixes and the bug persists, STOP guessing and create a temporary debug log.** This is the fastest way to pinpoint elusive bugs.

## Why This Works

- Theory-based fixes are **blind** — you're guessing what's wrong
- A debug log gives you **hard evidence** of exactly what's happening at runtime
- One test with a debug log often reveals the root cause **instantly**

## The Pattern

### 1. Create a Temporary Log Helper
```csharp
// Add to the class you're debugging
// NOTE: the real debug log lives at %LOCALAPPDATA%/PowerXKeys/debug_log.txt
// (AppConstants.DebugLogPath, PowerX.Core/Models/AppConstants.cs:24)
private static readonly string _debugLogPath = System.IO.Path.Combine(
    Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData), "PowerXKeys", "debug_log.txt");
private static void DebugLog(string msg)
{
    try { System.IO.File.AppendAllText(_debugLogPath, $"[{DateTime.Now:HH:mm:ss.fff}] {msg}\n"); } catch { }
}
```

### 2. Log at Critical Points
- **Object creation** — "Was this thing even created?"
- **Method entry** — "Did this method even get called?"
- **Condition branches** — "Which branch did we take?"
- **Parameter values** — "What were the actual values?"

### 3. Test & Read the Log
- Run the app
- Reproduce the bug
- Read the log file from `%LOCALAPPDATA%/PowerXKeys/debug_log.txt`
- The root cause is usually obvious from the log

### 4. Clean Up
- Remove ALL `DebugLog()` calls
- Remove the helper method
- Delete the log file from `%LOCALAPPDATA%/PowerXKeys/`
- Rebuild

## Real Example: Win+V Capture Bug (2026-06-16)

**Problem:** Win+V was captured by a test button but NOT by Quick Action cards.

**5+ theory-based fixes failed** (hook lifecycle, event subscription, AHK interference, etc.)

**Debug log revealed in 6 lines:**
```
[21:23:32.431] === NEW HOOK CREATED === advancedMode=True    ← only ONE hook created (test button)
[21:23:32.435] SetWindowsHookEx result: _kbHookId=364316141  ← hook installed OK
[21:23:32.806] KEY vk=0x5B msg=0x0100 down=True up=False     ← Win key captured
[21:23:33.141] KEY vk=0x56 msg=0x0100 down=True up=False     ← V key captured
[21:23:33.250] KEY vk=0x56 msg=0x0101 down=False up=True     ← V key released
```

**No second "NEW HOOK CREATED"** → Quick Action card never created a hook!

**Root cause:** Property ordering bug — `CurrentlyBindingAction = action` fired the event BEFORE `IsPathBinding = true` was set, so the handler saw `false` and skipped hook creation.

**Fix:** 2 lines swapped. Done in 30 seconds.

## Rules

> [!CAUTION]
> - **ALWAYS remove custom debug logs before committing** — do not leave temporary `DebugLog()` helper methods or custom logs in your commits.
> - **Leave built-in service loggers alone** — the permanent loggers in core services (like `MacroExecutionService.cs`) are meant to stay.
> - **Log to `%LOCALAPPDATA%/PowerXKeys/debug_log.txt`** for temporary files — the app's own `DebugLogger` already writes there (`DebugLogger.cs` → `AppConstants.DebugLogPath`).
> - **Use try/catch** — custom logging should never crash the app.
> - **Log timestamps with milliseconds** — helps spot timing/race issues.
> - **Log specific values** — don't just log "entered method", log the actual state.

---

## Important: Debug Logs Are ALREADY Baked In

The DebugLog helper is permanently added to MacroExecutionService.cs, MacroRecordingService.cs, and SmoothTraceEngine.cs. You do NOT need to add it - just READ THE FILE at `%LOCALAPPDATA%/PowerXKeys/debug_log.txt` after the user tests.

So the workflow is even simpler:
1. Ask user to reproduce the issue
2. Read C:\Users\Maaz\AppData\Local\PowerXKeys\debug_log.txt yourself
3. Pinpoint the bug from the log
4. Fix it

No adding debug code. No removing it after. Just read.

---

## Real Example: MouseTrace Cursor Snapping to Left Edge (2026-06-26)

**Problem:** During trace preview, cursor would snap to the far left edge of the screen before playing the trace. Happened after the first drag/click block.

**Debug log revealed in 1 line:**
[ExecuteStepAsync.MouseTrace] SetCursorPos to X=-199, Y=196 (offsets: 160, 428, resolvedWindow=True)

Negative X = cursor going off-screen! Root cause was instantly obvious: step.X was -359 (window-relative), window offset was 160, so -359 + 160 = -199. Wrong calculation when window moved between recording and playback.

**Fix:** Use AbsoluteX/Y (recorded screen coordinates) instead of computing window-relative + live offset. Done in 2 minutes.

---

## Future Idea: Toggleable Debug Logging for User Bug Reports

**Status:** Parked - discuss before implementing

**The Problem:**
The current DebugLog() is always ON and writes everything to `%LOCALAPPDATA%/PowerXKeys/debug_log.txt`. This is fine for development but NOT for shipped builds - it creates huge files, logs private data (window titles, coords), and confuses users.

**The Idea:**
Add a static flag - DebugLoggingEnabled = false by default. Wrap all DebugLog() calls behind it. Add a hidden way for users to enable it when reporting bugs (e.g. Shift+click the version number in Settings).

**User flow for bug reports:**
1. User hits a bug
2. You tell them: hold Shift and click the version number in Settings
3. They reproduce the bug
4. debug_log.txt appears in `%LOCALAPPDATA%/PowerXKeys/`
5. They attach it to their bug report and send to you
6. You read it and pinpoint the issue instantly - same as dev debugging

**Why it's better than removing logs entirely:**
- Zero impact on normal users
- Massive debugging power when you need it
- Users feel involved in fixing their own issue

**Files to change when implementing:**
- MacroExecutionService.cs - wrap DebugLog calls
- MacroRecordingService.cs - wrap DebugLog calls  
- SmoothTraceEngine.cs - wrap DebugLog calls
- Add toggle in SettingsDashboard (hidden, Shift+click version)
- ConfigManager - save the flag
