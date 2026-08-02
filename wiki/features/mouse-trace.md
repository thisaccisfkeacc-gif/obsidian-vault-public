---
tags: [feature, mouse-trace, smooth-trace, recording]
date: 2026-05-23
sources:
  - Services/SmoothTraceEngine.cs
status: current
---

# Mouse Trace 🖱️

Mouse Trace is the system for recording and replaying precise mouse movement paths. When a user records a macro with "Record Full Mouse Path" enabled, the cursor trajectory is saved as a trace file and replayed with sub-millisecond precision.

## Overview

- Records mouse position at ~20ms intervals during macro recording
- Trace data stored as CSV files: `x,y,timestamp`
- Three playback physics profiles for different use cases
- Uses C# P/Invoke (`SetCursorPos`) — NOT AHK — for smooth cursor control

## Recording

During macro recording, `MacroRecordingService` samples the cursor position and writes coordinates with exact millisecond timestamps (`_traceStartTimeMs`). The trace file is saved to:

```
%APPDATA%/PowerX_Keys/TraceData/{guid}.dat
```

Format per line: `x,y,timestampMs`

If no timestamp column exists, the engine assumes 20ms intervals.

### 2-State Post-Recording Cycle Button 🔄

The recording service always captures full trace data in the background. After recording completes, if mouse trace data was captured, a 2-state cycle button appears on the editor toolbar.

Clicking this button toggles between two views:
1. **No Trace (Default)**: standard click steps only.
2. **Full Trace**: displays standard click steps plus all captured mouse trace paths.

This allows the user to dynamically toggle between showing and hiding trace steps post-recording.

### Selected Path Visualization Overlay 🪰

When a user selects a `MouseTrace` step block in either the timeline steps list or the mini grid, the editor automatically renders a visual overlay of the entire mouse path directly on the screen.

- **Marching Ants Effect**: The path is drawn as a dashed, neon lavender/purple line animated with a marching ants scroll effect to show the direction of movement.
- **Start and End Markers**: Mint green circular marker highlights the **START** position, and coral red circular marker highlights the **END** position.
- **Relative Coordinates support**: If the step's coordinate mode is set to Window or Client, the overlay automatically resolves the target window boundaries and adjusts the drawn path to overlay accurately on top of that window.
- **Auto-Hide / Window Deactivation / Debounced Rendering**: The path overlay automatically hides when the step is deselected, when the editor is closed/unloaded, or when the user deactivates the application window (e.g. by Alt-Tabbing away). Overlay updates are debounced by 50ms and the overlay window is cached and hidden/shown (instead of closed/recreated) to prevent flickering and flashing during fast selection changes or mode toggles.
- **Minimized / Missing Window Safety**: If a step is window-relative but the target window is missing, minimized, or hidden, the visual path overlay is safely skipped (preventing coordinates from drawing off-screen at window virtualization locations like `-32000`).

## Playback Engine

`SmoothTraceEngine.PlayTraceAsync()` handles replay with:

### Three Physics Profiles

| Profile | ID | Behavior |
|---------|-----|----------|
| **Smooth** | 0 | Catmull-Rom spline interpolation (5 points per segment) |
| **Instant** | 1 | Direct point-to-point jumps (original data, no smoothing) |
| **Original** | 2 | Linear interpolation (5 points per segment) |


### Speed Multiplier

A `speedMultiplier` parameter scales all timestamps:
```
expectedElapsed = point.T / speedMultiplier
```
- `1.0` = original speed
- `2.0` = double speed
- `0.5` = half speed

### Precision Timing

The engine uses a three-tier waiting strategy for frame-accurate cursor positioning:

| Wait Remaining | Strategy |
|---------------|----------|
| >15ms | `Task.Delay(1)` — yields to thread pool |
| 2–15ms | `Thread.Sleep(1)` — kernel-level sleep |
| <2ms | `Thread.SpinWait(100)` — busy-wait for maximum precision |

### Jitter Prevention

If the thread is preempted (OS scheduler delay), the engine fast-forwards to the latest point it should be at, preventing a burst of instantaneous `SetCursorPos` calls.

## Spline Math

The **Smooth Curve** profile uses Catmull-Rom spline interpolation:

```
x = 0.5 * (2*p1 + (-p0+p2)*t + (2*p0-5*p1+4*p2-p3)*t² + (-p0+3*p1-3*p2+p3)*t³)
```

This produces natural, curved paths that look human-like even at high speed.

## Integration with Macros

- Mouse trace steps have `MacroStepType.MouseTrace`.
- The trace file ID is stored in `MacroStep.TraceFileId`, and the path resolves to `%APPDATA%/PowerX_Keys/TraceData/trace_{TraceFileId}.dat`.
- The macro editor timeline displays a premium timeline card for Mouse Trace steps containing:
  - A pulsing neon-green status dot indicating an active trace.
  - A start coordinate pill (with green target icon).
  - A transition arrow (`→`).
  - An end coordinate pill (with red target icon).
- Start (`X`, `Y`) and end (`EndX`, `EndY`) coordinates are automatically resolved and populated from the `.dat` trace file upon step deserialization/loading or property changes. Screen-relative `AbsoluteX` and `AbsoluteY` properties are captured during recording to ensure floating widget containment checks succeed even when relative coordinate modes are active.
- `MacroTransferManager` bundles `.dat` trace files into export packages.
- Coordinate scaling is applied during import based on screen resolution differences.

### Per-Step Mouse Path Delays ⏳
- Timeline steps of type `MouseTrace` (mouse path steps) feature a settings gear icon.
- Clicking the gear icon opens a settings popup to customize:
  - **Hold Delay (ms)**: Wait time after clicking down at the start coordinate before beginning cursor movement (default 0ms).
  - **Release Delay (ms)**: Wait time after arriving at the end coordinate before releasing the mouse click (default 0ms).
- Both properties support dropdown preset values (0ms, 50ms, 100ms, 200ms, 500ms, 1000ms) and a "Custom..." selector that opens a numeric input box for arbitrary millisecond values.
- Supported locally in the playback loop and compiled directly as `Sleep(ms)` statements in generated AutoHotkey scripts.

## Smart Playback & Instant Conversion Utility ⚡

### Smart Playback System
- Automatically optimizes execution of recorded mouse drags depending on the complexity profile:
  - **Simple Drags**: Automatically flattened into direct linear drag-and-drops with adaptive interpolation steps: `steps = Clamp(distance/30, 3, 25)`.
  - **Complex Drags**: Retains full curves/wiggles for accuracy.
  - **Raw Human Mode**: Bypasses any auto-cleaning to preserve hand movements exactly as recorded.
- Caches the complexity assessment (`Simple` vs `Complex`) on the `MacroStep` object (`DragComplexity`) to avoid disk read overhead during playback.

## Key Files


- [SmoothTraceEngine.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Services/SmoothTraceEngine.cs) — Replay playback engine
- [MacroRecordingService.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Services/MacroRecordingService.cs) — Recording logic
- [MousePathOverlayWindow.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/MousePathOverlayWindow.cs) — Path visualization on selection
- [DragComplexityAnalyzer.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Services/DragComplexityAnalyzer.cs) — Linearity analyzer for mouse paths

## Related Pages

- [[macro-editor]]
- [[macro-item]]
- [[macro-transfer-manager]]
- [[Smart_Playback_Spec]]
