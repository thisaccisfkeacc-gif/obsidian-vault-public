# Block Audit: MouseTrace (Deep — Cross-Path)

**Date:** 2026-07-24
**Scope:** C# in-process, AHK compiled, AHK preview (SingleStep)

---

## SUMMARY

MouseTrace block replays recorded mouse movement paths with spline interpolation. Uses CSV trace files (x,y,timestamp). Three physics profiles: Smooth (Catmull-Rom), Instant (teleport), Original (linear).

---

## BUGS

### 1. Right-drag always uses left button (FIXED)
- **Severity:** Critical (fixed)
- **Location:** MacroExecutionService.cs:1816, ScriptCompilerService.cs:2668
- **Problem:** Both C# and AHK hardcode `LEFTDOWN`/`Click("Down")` for drag traces. Right-drag steps incorrectly held down the left mouse button.
- **Fix:** Inspect step button type, use `MOUSEEVENTF_RIGHTDOWN`/`Click("Right Down")` for right-drag.

### 2. AHK Instant mode replays full path instead of teleporting
- **Severity:** Medium
- **Location:** ScriptCompilerService.cs:2606-2643
- **Problem:** For physicsProfile=1 (Instant), AHK still reads every CSV line and moves point-by-point. C# teleports to end instantly. Behavior differs between paths.

### 3. AHK MouseTrace ignores coordinate offsets for trace points
- **Severity:** Medium
- **Location:** ScriptCompilerService.cs:2676
- **Problem:** Start position gets window offset applied, but raw CSV coordinates in playback loop do not. C# applies offset to every point in SmoothTraceEngine.

---

## DEAD CODE

1. `DragComplexity` property — only consumed in C# execution, serialized but unused in AHK paths.
2. `AbsoluteX`/`AbsoluteY`/`AbsoluteEndX`/`AbsoluteEndY` — `[JsonIgnore]`, set during recording, never consumed.

---

## REDUNDANCIES

1. `MousePathOverlayWindow` duplicates `SmoothTraceEngine`'s trace parsing, smoothing, and `PointD` struct. Should be consolidated.
2. AHK compiled has TWO separate drag implementations (MouseClick.DragAndDrop vs MouseTrace drag).

---

## MISSING FEATURES

| Feature | C# | AHK Full | AHK Preview |
|---|---|---|---|
| Trace playback | ✅ | ✅ | ❌ (uses native API) |
| Instant teleport | ✅ | ❌ (replays) | N/A |
| Coordinate offsets | ✅ | ❌ | N/A |
| HoldDelay/ReleaseDelay | ✅ | ✅ | N/A |
| Physics profiles | ✅ | ✅ (partially) | N/A |
| Right-drag | ✅ (after fix) | ✅ (after fix) | N/A |

---

## INCONSISTENCIES

1. C# SmoothTraceEngine uses `timeBeginPeriod(1)` for high-resolution timing. AHK uses `Sleep()` with ~15ms granularity. AHK trace playback is less smooth.
2. C# uses `SetCursorPos` + `mouse_event`. AHK uses `Click`/`MouseMove`. Different OS-level mouse event pipelines.

---

## VERIFIED OK

- Trace file CSV format — consistent across all paths
- MouseTrace start/end point resolution — correct
- Recording state management — proper
- Speed multiplier — applied correctly in C#, partially in AHK
