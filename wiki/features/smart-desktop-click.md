# Smart Desktop Click Handling — Feature Proposal

> **Status:** Idea / Not Implemented  
> **Priority:** Enhancement  
> **Category:** Playback Intelligence

---

## Problem

When a user records a macro that includes clicking an app icon on the **Desktop**, and during playback other windows are covering the desktop:

1. The `WindowAction: Activate` step tries `WinActivate` on Desktop/Explorer
2. But the Desktop is a **special window** — it's always behind all other windows
3. `WinActivate` on Desktop does NOT minimize other apps
4. The click goes to absolute coordinates → hits the wrong window instead of the desktop icon

**Result:** The macro fails silently — it clicks whatever window is on top, not the desktop icon.

---

## Competitor Analysis

| Tool | Handles Desktop Clicks? | How? |
|------|------------------------|------|
| **Macro Recorder (Bartels Media)** | ❌ No | Has `FixWindowRect` for app windows, but no special Desktop handling |
| **AutoHotkey** | ❌ Manual | User must manually add `Send, #d` or `WinActivate, ahk_class Progman` |
| **Other macro recorders** | ❌ No | None auto-handle Desktop as a special case |

**No major macro recorder handles this automatically.** This is an opportunity to differentiate.

---

## Proposed Solution

### During Recording
- Detect when the foreground window is the **Desktop** by checking:
  - Window class = `Progman` (Program Manager)
  - Window class = `WorkerW` (alternative Desktop host when wallpaper slideshow is active)
  - Window title contains `Program Manager`
- Tag the `WindowAction: Activate` step with a flag like `IsDesktopWindow = true`

### During Playback (ScriptCompilerService)
- When compiling a `WindowAction: Activate` step that targets the Desktop:
  - Instead of `WinActivate`, emit `Send("#d")` (Win+D = Show Desktop)
  - OR use COM call: `ComObjCreate("Shell.Application").MinimizeAll()`
  - Add a small delay (300-500ms) for the minimize animation to complete
  - Then proceed with the click at the recorded coordinates

### For Regular App Windows (Bonus Improvement)
- Use the **Alt key hack** before `SetForegroundWindow` to bypass Windows foreground lock:
  ```
  ; AHK pattern
  Send("{Alt}")  ; Trick Windows into allowing foreground change
  WinActivate("target window")
  WinWaitActive("target window",, 2)
  ```

---

## Technical Details

### Desktop Window Hierarchy (Windows)
```
Progman (Program Manager)
  └── SHELLDLL_DefView
       └── SysListView32 (the actual icon list view)

WorkerW (alternative host — active with wallpaper slideshow)
  └── SHELLDLL_DefView
       └── SysListView32
```

### Best Methods to Show Desktop (Ranked)
1. **`Shell.Application.MinimizeAll()`** — COM call, most reliable, no toggle behavior
2. **`Send("#d")`** — Win+D shortcut, but it's a toggle (pressing again restores windows)
3. **`WinActivate("ahk_class Progman")`** — may not fully clear foreground windows

### Detection Code (C# — for recording)
```csharp
[DllImport("user32.dll")]
static extern int GetClassName(IntPtr hWnd, StringBuilder lpClassName, int nMaxCount);

bool IsDesktopWindow(IntPtr hwnd)
{
    var className = new StringBuilder(256);
    GetClassName(hwnd, className, className.Capacity);
    string cls = className.ToString();
    return cls == "Progman" || cls == "WorkerW";
}
```

### Compiled AHK Output (for playback)
```ahk
; Instead of: WinActivate("Program Manager ahk_exe explorer.exe")
; Emit:
ComObjCreate("Shell.Application").MinimizeAll()
Sleep(400)
Click, 150, 300  ; Desktop icon coordinates
```

---

## Impact
- **User benefit:** Macros that click desktop icons "just work" even when other apps are open
- **Competitive edge:** No competitor does this automatically
- **Risk:** Low — only affects Desktop-targeted steps, doesn't change behavior for regular windows

---

## Agent Review — Implementation Agent (2026-06-29)

**Verdict: ✅ Agree — solid proposal, ready to implement when prioritized.**

### What I Like
- Detection logic is correct — `Progman` + `WorkerW` covers all Windows versions
- COM `MinimizeAll()` is the right call — `Win+D` toggle behavior would break repeat runs
- Tagging with `IsDesktopWindow = true` during recording is clean — no runtime detection needed
- The Alt key hack for regular windows is a nice bonus improvement

### ⚠️ One Edge Case to Handle

`Shell.Application.MinimizeAll()` minimizes **ALL** windows — including PowerX Keys itself if it's visible. During macro playback, the app window could get minimized mid-execution.

**Mitigation:** After `MinimizeAll()`, restore PowerX Keys if it was visible:
```ahk
; Save our window state
wasVisible := WinExist("PowerX Keys")
ComObjCreate("Shell.Application").MinimizeAll()
Sleep(400)
; Restore ourselves if needed (minimized to tray is fine)
if (wasVisible)
    WinRestore("PowerX Keys")
```

Or simpler: since macros run from tray/hotkey anyway, PowerX Keys is usually already minimized. This may not be an issue in practice — just flag it for testing.

### Priority Opinion
This is a **nice-to-have**, not urgent. Desktop icon clicking is a niche use case. Focus on the core Smart Playback System first, then circle back to this.

