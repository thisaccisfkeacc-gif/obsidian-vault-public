# Block Audit: WindowAction (Deep — Cross-Path)

**Date:** 2026-07-24
**Scope:** C# in-process, AHK compiled, AHK preview (SingleStep)

---

## Summary

WindowAction block manages windows — activate, close, minimize, maximize, restore, wait, desktop click, browser tab switching.

---

## BUGS

### 1. AHK auto-launch missing security blocklist
- **Severity:** Medium
- **Location:** ScriptCompilerService.cs
- **Problem:** When launching apps via WindowAction, C# checks a security blocklist (blocks critical system executables). AHK compiled path does NOT check the blocklist — it launches whatever path is specified without filtering.

### 2. Browser tab switching not in C#
- **Severity:** Low (feature gap)
- **Location:** MacroExecutionService.cs
- **Problem:** Browser tab switching (Chrome, Edge, Firefox) via `ControlSend` is only implemented in AHK. C# execution silently skips browser tab steps.

### 3. ComObjCreate AHK v1 syntax (FIXED)
- **Severity:** Critical (fixed)
- **Location:** ScriptCompilerService.cs — Smart Desktop Click
- **Problem:** `ComObjCreate("Shell.Application").MinimizeAll()` — v1 syntax. Fixed to `ComObject("Shell.Application").MinimizeAll()`.

---

## DEAD CODE

1. `Restore` remapping — Some WindowAction restore steps get remapped to `Maximize` internally. The separate `Restore` action in the C# path is partially dead.

---

## REDUNDANCIES

1. `CaptureInteractiveWindow` methods in WindowCaptureService are 80% identical between the two capture modes (window pick vs process pick). Should be consolidated.

---

## MISSING FEATURES

| Feature | C# | AHK Full | AHK Preview |
|---|---|---|---|
| Security blocklist | ✅ | ❌ | N/A |
| Browser tab switching | ❌ | ✅ | N/A |
| Desktop click | ✅ | ✅ (after fix) | ✅ |
| Wait for window | ✅ | ✅ | ✅ |

---

## VERIFIED OK

- Window activate/close/minimize/maximize — consistent across paths
- Coordinate mode for window-relative actions — consistent
- Timeout handling for Wait — consistent
