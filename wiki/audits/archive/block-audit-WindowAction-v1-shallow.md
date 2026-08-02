# Block Audit: WindowAction

## Summary
WindowAction handles window activation (with 5-step escalation), Move, Close, Minimize, Maximize. The Activate action is the most complex with browser tab switching, desktop detection, auto-launch, and SearchOtherWindows cascading.

---

### [SEVERITY: Critical] — ComObjCreate is AHK v1 syntax (Desktop Click broken)
**Scenario:** User has a WindowAction block targeting the Desktop (IsDesktopWindow=true). The macro fires the Smart Desktop Click.
**Impact:** The generated AHK uses `ComObjCreate("Shell.Application").MinimizeAll()` — but `ComObjCreate` does not exist in AHK v2. The correct function is `ComObject()`. At runtime, AHK v2 throws "Unknown function" error. The `try/catch` fallback catches this and uses `Send("#d")` instead, so the feature still works but via a slower path.
**Verified:** Yes — `ComObjCreate` is AHK v1 only. GOTCHAS.md G-001 explicitly warns about this.
**Fixed:** Yes — changed to `ComObject("Shell.Application").MinimizeAll()`

### [SEVERITY: Low] — Auto-prepend ahk_exe for bare .exe WindowTitle
**Scenario:** User sets WindowTitle to "chrome.exe" without the "ahk_exe" prefix.
**Impact:** Code: `if (!winTitleRaw.StartsWith("ahk_", ...) && winTitleRaw.EndsWith(".exe", ...)) winTitleRaw = "ahk_exe " + winTitleRaw`. Correct auto-fix behavior.
**Verified:** Yes
**Fixed:** N/A

### [SEVERITY: Low] — "Restore" action merged into "Activate"
**Scenario:** Old macros with Value="Restore" from V1 import.
**Impact:** Code: `if (action == "Restore") action = "Activate"`. This means "Restore" follows the full 5-step activation cascade (which includes restore from minimized). Correct backward compat.
**Verified:** Yes
**Fixed:** N/A

### [SEVERITY: Low] — Tab switching uses UIA ComObject with hardcoded CLSIDs
**Scenario:** Browser tab switching via UI Automation in the Activate cascade.
**Impact:** Uses COM CLSID `{ff48dba4-60ef-4201-aa87-54103eef594e}` (CUIAutomation) and IID `{30cbe57d-d9d0-452a-ab13-7ac5ac4825ee}` (IUIAutomation). These are stable Windows system GUIDs that won't change. The tab matching strips browser suffixes and notification counts for fuzzy matching.
**Verified:** Yes
**Fixed:** N/A

### [SEVERITY: Low] — WinSetAlwaysOnTop(1) then WinSetAlwaysOnTop(0) pattern
**Scenario:** Window activation uses temporary AlwaysOnTop to force window to front.
**Impact:** This is a common workaround for Windows focus-stealing prevention. Sets AOT, activates, then removes AOT. Correct pattern.
**Verified:** Yes
**Fixed:** N/A

### [SEVERITY: Low] — Move action doesn't activate window
**Scenario:** User has a Move action (reposition/resize window).
**Impact:** Compilation: `WinMove(x, y, w, h, title)` without `WinActivate`. This is correct — "Move" should only reposition, not bring to front.
**Verified:** Yes
**Fixed:** N/A

### [SEVERITY: Low] — SmartWait adds 10s timeout before activation cascade
**Scenario:** User enables SmartWait on a WindowAction block.
**Impact:** Emits `WinWait("{winTitle}", , 10)` before the cascade. If the window doesn't appear in 10s, the cascade proceeds anyway (WinWait just returns). No harm — the cascade handles "window not found" gracefully.
**Verified:** Yes
**Fixed:** N/A

---

## Verdict

One critical bug found and fixed: `ComObjCreate` → `ComObject` (AHK v1 vs v2 syntax). The fallback to `Send("#d")` meant the Desktop Click feature still worked, but via a suboptimal path. Now fixed to use the correct AHK v2 COM syntax.
