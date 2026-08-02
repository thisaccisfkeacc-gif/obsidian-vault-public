---
tags: [service, mouse, animation, spline, physics]
date: 2026-05-23
sources:
  - Services/SmoothTraceEngine.cs
status: complete
---

# Smooth Trace Engine

Replays recorded mouse movement traces with **human-like cursor motion** using Catmull-Rom spline interpolation. Used during both C# macro playback (Preview) and compiled AutoHotkey (AHK) playback (Production) to reproduce natural-looking mouse paths.

## Purpose

- Plays back mouse traces recorded by [[macro-recording]]
- Supports three physics profiles for different realism levels across both preview and compiled modes
- High-precision timing with adaptive wait strategies
- Speed multiplier support for fast/slow playback

## Physics Profiles

| Profile | ID | Algorithm | Use Case |
|---------|-----|-----------|----------|
| **Smooth** | 0 | Catmull-Rom spline | Most human-like, curves through control points |
| **Instant** | 1 | No interpolation | Exact recorded points, jumpy movement |
| **Original** | 2 | Linear interpolation | Straight-line segments between points |


## Trace File Format

Files are stored as `.dat` in `%APPDATA%\PowerX_Keys\TraceData\`:

```
x,y,timeOffsetMs
500,300,0
505,302,20
510,308,40
...
```

- Default assumes 20ms intervals if time column is missing
- Parsed by `ParseTrace()` into `List<PointD>` structs

## Catmull-Rom Spline (Profile 0)

The `GenerateSmoothPath()` method generates 5 interpolated points per raw segment using the Catmull-Rom formula:

```
x = 0.5 * ((2·P1) + (-P0 + P2)·t + (2P0 - 5P1 + 4P2 - P3)·t² + (-P0 + 3P1 - 3P2 + P3)·t³)
```

- Uses 4 control points (P0, P1, P2, P3) per segment
- Boundary handling: duplicates first/last points at edges
- Time is linearly interpolated between segments

## Linear Interpolation (Profile 2)

`GenerateLinearPath()` generates 5 evenly-spaced points per segment using simple lerp:

```
x = P1.x + (P2.x - P1.x) * t
```

## Playback Engine

`PlayTraceAsync()` runs on a background thread with a `Stopwatch`:

1. Sets initial cursor position immediately
2. Loops through smoothed path points
3. **Fast-forward fix** — if the thread was preempted, skips to the latest point we should be at (prevents burst of instantaneous `SetCursorPos` calls)
4. Uses adaptive waiting:
   - `>15ms` remaining → `Task.Delay(1)` (yields to thread pool)
   - `>2ms` remaining → `Thread.Sleep(1)` (OS-level sleep)
   - `<2ms` remaining → `Thread.SpinWait(100)` (micro-spin for high precision)

## Speed Control

- `speedMultiplier` divides all time offsets: `expectedElapsed = point.T / speedMultiplier`
- `2.0x` = double speed, `0.5x` = half speed

## Compiled AHK Playback

During macro compilation on the dashboard (when clicking the **START** button), `ScriptCompilerService` reads the chosen `MousePhysicsProfile`. If set to **Smooth** (0) or **Original** (2), it pre-processes the interpolated coordinates on the C# side at compile-time and saves them to a profile-specific cached file:
`trace_{id}_profile_{profile}.dat`

The generated AutoHotkey (AHK) script is then compiled to read and replay the coordinates from this profile-specific trace file rather than the raw trace file, matching the desired physics feel in production.

## Key Methods

| Method | Description |
|--------|-------------|
| `PlayTraceAsync()` | Main entry — loads trace, smooths, plays back |
| `ParseTrace()` | Reads `.dat` file into `PointD` list |
| `GenerateSmoothPath()` | Catmull-Rom spline interpolation |
| `GenerateLinearPath()` | Simple linear interpolation |

## Key Files

- [SmoothTraceEngine.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Services/SmoothTraceEngine.cs) — 177 lines

## Related Pages

- [[macro-recording]]
- [[macro-execution]]
- [[script-compiler]]
