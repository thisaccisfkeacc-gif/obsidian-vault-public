# Block Audit: MouseClick (Deep — Cross-Path)

**Date:** 2026-07-24
**Scope:** C# in-process, AHK compiled, AHK preview (SingleStep)

---

## Summary

MouseClick block handles all mouse actions — click, right-click, middle-click, drag-and-drop, scroll, hold/release, multiple clicks, timer clicks. 10+ action types, 6 target modes, coordinate modes.

---

## BUGS

### 1. AHK Preview doesn't handle MouseClick
- **Severity:** By design (not a bug)
- **Location:** ScriptCompilerService.SingleStep.cs
- **Problem:** MouseClick preview uses native C# win32 API (SetCursorPos/mouse_event) directly, not AHK. This is intentional for performance.

### 2. AHK drag-and-drop is a teleport, not smooth
- **Severity:** By design (not a bug)
- **Location:** ScriptCompilerService.cs:1918-1980
- **Problem:** AHK generates `Click(Down) → Sleep(50) → Click(Up)` — instant teleport. C# uses SmoothTraceEngine for smooth drags. This is intentional — AHK prioritizes automation speed, C# prioritizes humanized recording playback.

### 3. MousePhysicsProfile source differs
- **Severity:** Low (design choice)
- **Location:** MacroExecutionService.cs:1174 vs ScriptCompilerService.cs
- **Problem:** C# uses global `ConfigManager.Current.Settings.MousePhysicsProfile`. AHK uses per-macro `targetMacro.MousePhysicsProfile`. Can cause different behavior.

---

## DEAD CODE

1. `"Double"` click detection — code checks `actionStr.Contains("Double")` but no MouseAction in the UI dropdown contains "Double". Unreachable. "Multiple Clicks" with ClickCount=2 achieves double-click instead.

---

## REDUNDANCIES

1. Two different drag flag computations — MouseClick handles right-drag properly (`"Right Drag and Drop"`), but the drag path has redundant flag checks.

---

## MISSING FEATURES

| Feature | C# | AHK Full | AHK Preview |
|---|---|---|---|
| All click types | ✅ | ✅ | ✅ (native API) |
| Smooth drag | ✅ | ❌ (teleport) | ✅ (native API) |
| DragComplexity analysis | ✅ | ❌ | N/A |
| Hold/Release delays | ❌ (fixed 50ms) | ❌ (fixed 50ms) | N/A |
| Right-drag | ✅ | ✅ | ✅ |
| Humanization | ✅ | ✅ | N/A |

---

## VERIFIED OK

- Click actions (Left/Right/Middle) — consistent across paths
- Scroll (Up/Down/amount) — consistent
- Multiple clicks with interval — consistent
- Coordinate modes (absolute/relative/client) — consistent
- Hold/Release for keyboard-style mouse hold — consistent
- Humanization (±2px jitter) — consistent
