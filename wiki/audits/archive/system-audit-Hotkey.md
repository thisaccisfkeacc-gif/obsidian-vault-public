# System Audit: Hotkey Service (Deep)

**Date:** 2026-07-24
**Scope:** HotKeyService, HotkeyCaptureHook, hotkey compilation

---

## BUGS

### 1. Win key guard only checks LWin, not RWin
- **Severity:** Low
- **Location:** ScriptCompilerService.cs:5131
- **Problem:** `GetKeyState("LWin", "P")` only checks left Win key. Hotkey could fire with RWin held. Ctrl/Alt/Shift check both L/R variants.

### 2. `HotKeyPressed` event never subscribed
- **Severity:** Info (dead infrastructure)
- **Location:** HotKeyService.cs:27,81-90
- **Problem:** Event declared and invoked but no code subscribes. Real hotkey system uses AHK engine. WM_HOTKEY handling is dead.

### 3. AHK key capture system fully dead
- **Severity:** Info (dead code)
- **Location:** ScriptLibraryView.xaml.cs:19-22,204-355
- **Problem:** ~150 lines of vestigial AHK capture code. Explicitly commented out: "C# hook now handles ALL keys".

---

## DEAD CODE

1. AHK key capture fields and methods — ~150 lines
2. `HotKeyPressed` event + WM_HOTKEY handler
3. `TriggerKey` and `TriggerModeString` on MacroItem — Experimental AI fields not connected to real pipeline

---

## INCONSISTENCIES

1. Hotkey storage format mismatch — Legacy mode produces WPF key names, advanced mode produces AHK-friendly names
2. Hold/LongPress quick-tap passthrough doesn't include modifiers — `SendEvent("{cleanKey}")` fires bare key without modifier context

---

## VERIFIED OK

- Hook lifecycle — properly installed/uninstalled, IDisposable
- Hook re-initialization — removes old hook before adding new
- Modifier state tracking — correct in both legacy and advanced modes
- Thread safety — callbacks marshaled to UI thread
- DoubleTap, Hold/LongPress, Toggle, PressAndRelease, Release compilation — all correct
- Conflict detection — correct with profile scoping
- Mouse trigger swallowing — correct
- Cancel button passthrough — correct
