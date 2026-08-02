# Block Audit: Delay (Deep — Cross-Path)

**Date:** 2026-07-24
**Scope:** C# in-process, AHK compiled, AHK preview (SingleStep)

---

## BUGS

### 1. Sandbox compiler ignores PlaybackSpeed
- **Severity:** Medium
- **Location:** ScriptCompilerService.cs:4864
- **Problem:** Sandbox uses raw `Sleep({step.Duration.Value})` without dividing by PlaybackSpeed. Main compiler correctly does `Math.Max(10, step.Duration.Value / targetMacro.PlaybackSpeed)`.

### 2. Minimum delay floor inconsistency — C# 1ms vs AHK 10ms
- **Severity:** Low
- **Location:** MacroExecutionService.cs:1264 vs ScriptCompilerService.cs:1829
- **Problem:** AHK enforces `Math.Max(10, ...)` (10ms minimum). C# enforces `Math.Max(1, d)` (1ms minimum). At high PlaybackSpeed (5x), a 50ms delay compiles to `Sleep(10)` in AHK but `SafeDelay(1)` in C# — 10x faster.

### 3. Division-by-zero on PlaybackSpeed=0
- **Severity:** Low
- **Location:** MacroExecutionService.cs:1263
- **Problem:** No guard against `multiplier == 0`. `(int)(step.Duration.Value / 0.0)` produces `Infinity` cast to `int`. UI prevents this but code has no safety clamp.

---

## DEAD CODE

1. SingleStep.cs — No handler for `MacroStepType.Delay`. Preview produces empty AHK script.

---

## VERIFIED OK

- C# uses `Task.Delay()` (async-safe)
- AHK uses `Sleep()` (correct)
- PlaybackSpeed division applied in main compiler
- Humanization consistent between C# and AHK
