# System Audit: Recording System (Deep)

**Date:** 2026-07-24
**Scope:** MacroRecordingService, recording state management, trace capture

---

## BUGS

### 1. Trace-off drag missing EndX/EndY
- **Severity:** Medium
- **Location:** MacroRecordingService.cs:889-914
- **Problem:** When `_traceCaptureMode == 0` and a drag occurs, a minimal MouseTrace step is created with X/Y set but EndX/EndY never assigned. Trace file saved to wrong location (`BaseDirectory/Traces/` instead of `%APPDATA%/PowerX_Keys/TraceData/`). `EnsureTraceStartEndPoints` won't find it. Drag endpoint permanently lost.

### 2. Pending window focus never flushed after drag
- **Severity:** Medium
- **Location:** MacroRecordingService.cs:818-961
- **Problem:** `FlushPendingWindowFocus()` NOT called in MouseUp handler. Window focus events suppressed during `_isTracing` stay pending forever after drag ends. `_activeWindowTitle`/`_activeWindowX`/`_activeWindowY` become stale.

### 3. AHK output reader potential deadlock
- **Severity:** Low
- **Location:** MacroRecordingService.cs:560-570
- **Problem:** `DispatchEvent` calls `Dispatcher.Invoke()` (synchronous) when `UseAsyncMacroRecording` is false. If UI thread waits on AHK thread during shutdown, deadlock possible. Fix: use `BeginInvoke`.

### 4. Widget-release filter too broad
- **Severity:** Low
- **Location:** MacroEditorViewModel.Recording.cs:532
- **Problem:** `step.Value.Contains("Up")` matches any step whose Value contains "Up". Fragile filtering logic.

### 5. Double-click threshold vs drag threshold mismatch
- **Severity:** Low
- **Location:** MacroRecordingService.cs:935-957
- **Problem:** Double-click detection uses 20px threshold, drag detection uses 4px. 5-19px movement between clicks classified as single click then bundled as double-click.

---

## DEAD CODE

1. `MacroEditorViewModel.Commands.cs.bak` — Backup file left in project.

---

## INCONSISTENCIES

1. Trace file location split — Full traces: `%APPDATA%/PowerX_Keys/TraceData/`. Minimal drag traces: `BaseDirectory/Traces/`. Two directories, two naming conventions.
2. `_isRecordingActive` vs `IsRecording` dual state — Set at different times in different error paths. Window where `_isRecording == false` but `_isRecordingActive == true`.
3. Right-click double-click produces "Multiple Clicks" instead of "Double Right Click".

---

## VERIFIED OK

- `_isTracing` lifecycle — properly managed
- Kill switch (double-escape) — correct
- Modifier auto-repeat filter — correct
- Smart delay cap — 30 seconds max
- Widget click stripping — correct with DPI scaling
- Dual-mode recording — NoTrace and AllTrace maintained
- Undo snapshot before recording — enables single Ctrl+Z undo
