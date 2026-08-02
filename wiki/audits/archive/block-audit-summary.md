# PowerX Keys V2 — Full Audit Summary

**Date:** 2026-07-24  
**Auditor:** Deep cross-path audit (C# vs AHK Full vs AHK Preview)  
**Project:** PowerX Keys V2  
**Scope:** All 21 block types + 4 core systems

---

## Block Audits (21 blocks)

| # | Block | Bugs | Severity | Status |
|---|-------|------|----------|--------|
| 1 | UIElement | 1 | Critical | ✅ Fixed |
| 2 | ImageSearch | 3 | Medium-Low | ⚠️ Noted |
| 3 | PixelSearch | 2 | Medium-Low | ⚠️ 1 Fixed |
| 4 | WindowAction | 3 | Medium-Critical | ⚠️ 1 Fixed |
| 5 | LogicIf | 2 | Critical-Low | ✅ 1 Fixed |
| 6 | WaitUntil | 3 | Medium | ⚠️ Noted |
| 7 | MouseClick | 3 | By Design | ✅ No action |
| 8 | MouseTrace | 3 | Critical-Medium | ✅ 1 Fixed |
| 9 | Keyboard | 3 | High-Medium-Low | ⚠️ Noted |
| 10 | Text | 5 | Medium-Low | ⚠️ Noted |
| 11 | Delay | 3 | Medium-Low | ⚠️ Noted |
| 12 | LoopSequence | 2 | High-Low | ⚠️ Noted |
| 13 | GroupHeader | 0 | — | ✅ Clean |
| 14 | Popup | 2 | Low | ✅ By Design |
| 15 | Notification | 3 | Medium-Low | ⚠️ Noted |
| 16 | SystemSound | 3 | Medium-Low | ⚠️ Noted |
| 17 | UserInput | 4 | Medium-Low | ⚠️ Noted |
| 18 | WaitForKey | 4 | High-Medium | ⚠️ Noted |
| 19 | FileLauncher | 3 | Medium-Low | ⚠️ Noted |
| 20 | CallMacro | 3 | Medium-Low | ⚠️ Noted |
| 21 | SetVariable | 2 | Medium-Low | ⚠️ Noted |

**Block Total: 58 findings (7 fixed, 28 medium, 23 low/by-design)**

---

## System Audits (4 systems)

| # | System | Bugs | Severity | Status |
|---|--------|------|----------|--------|
| 1 | Recording | 5 | Medium-Low | ⚠️ Noted |
| 2 | Capture | 5 | Medium-Low | ⚠️ Noted |
| 3 | Undo/Redo | 7 | Medium-Low | ⚠️ Noted |
| 4 | Hotkey | 3 | Low-Info | ⚠️ Noted |

**System Total: 20 findings (0 fixed, 8 medium, 12 low/info)**

---

## Grand Total: 78 findings across 25 components

---

## Priority Fixes (Confirmed by User)

### From Block Audits
1. **CallMacro stack ID-vs-Name** — ScriptCompilerService.cs:1003: Change `targetMacro.Id.ToString()` → `targetMacro.Name`
2. **Keyboard SafeSendKeys** — MacroExecutionService.cs:1409: Wrap single special key names in braces for .NET SendKeys

### From System Audits
3. **Recording: AHK output reader deadlock** — MacroRecordingService.cs: Change `Dispatcher.Invoke()` → `Dispatcher.BeginInvoke()`
4. **Recording: Trace-off drag EndX/EndY** — MacroRecordingService.cs:889-914: Set EndX/EndY from mouse-up position
5. **Capture: Multi-monitor DPI double-scaling** — CoordinatePickerWindow.xaml.cs:59-66: Check if transform already accounts for DPI
6. **Capture: `_navLevelOffset` thread race** — UIElementCaptureService.cs: Add lock/Interlocked
7. **Undo/Redo: DispatcherTimer.Tick leak** — UndoRedoService.cs:175-199: Unsubscribe in StopWatching()
8. **Undo/Redo: StepsEqual incomplete** — UndoRedoService.cs:120-127: Expand to compare all user-editable fields
9. **Hotkey: RWin guard** — ScriptCompilerService.cs:5131: Add RWin check alongside LWin

---

## SingleStep Preview Coverage

Only 4 of 21 blocks have AHK preview scripts (ImageSearch, PixelSearch, WindowAction, UIElement). All others use C# native execution for preview.

---

## Dead Code Summary

| Category | Items | Total Lines |
|----------|-------|-------------|
| Block audit dead code | 8 items | ~50 lines |
| Capture system dead code | 5 items | ~95 lines |
| Hotkey dead code | 3 items | ~150 lines |
| Undo/Redo dead code | 2 items | ~20 lines |
| Recording dead code | 1 item | Backup file |

**Total dead code identified: ~315 lines across 19 items**

---

## Files Modified

- 21 block audit files (deep, cross-path)
- 4 system audit files (deep)
- 1 summary file
- 11 old shallow audits archived to `archive/`
