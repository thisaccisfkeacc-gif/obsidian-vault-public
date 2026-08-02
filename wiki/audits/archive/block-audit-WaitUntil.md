# Block Audit: WaitUntil (Deep — Cross-Path)

**Date:** 2026-07-24
**Scope:** C# in-process, AHK compiled, AHK preview (SingleStep)

---

## Summary

WaitUntil block polls a condition (ImageFound, PixelFound, WindowExists, UIElementFound) until true or timeout. Has two condition types: WaitForImage and WaitUntilWindow.

---

## BUGS

### 1. Timeout mismatch — C# 5s vs AHK 10s
- **Severity:** Medium
- **Location:** MacroExecutionService.cs vs ScriptCompilerService.cs
- **Problem:** C# defaults to 5000ms timeout. AHK defaults to 10000ms. The same WaitUntil step behaves differently depending on execution path — C# times out 2x faster.

### 2. C# spawns new AHK process per poll iteration
- **Severity:** Medium
- **Location:** MacroExecutionService.cs
- **Problem:** For image/pixel conditions, C# launches a new AHK process for EACH poll attempt (up to timeout). This is expensive — process spawn overhead on every iteration. AHK compiled path runs a single loop with `Sleep`.

### 3. WindowActive check is wrong
- **Severity:** Medium
- **Location:** MacroExecutionService.cs
- **Problem:** `WindowActive` condition checks if the target window exists in ALL processes, not if it's the foreground window. A background window passes the check.

---

## DEAD CODE

1. None significant.

---

## REDUNDANCIES

1. Condition evaluation duplicated between C# and AHK (same as LogicIf).

---

## MISSING FEATURES

| Feature | C# | AHK Full | AHK Preview |
|---|---|---|---|
| All condition types | ✅ | ✅ | ✅ |
| Configurable timeout | ✅ (default 5s) | ✅ (default 10s) | ✅ |
| Poll interval | ✅ (100ms) | ✅ (configurable) | ✅ |

---

## VERIFIED OK

- Condition evaluation for ImageFound, PixelFound — consistent
- Timeout behavior — both paths stop polling at timeout
- Continue/Abort on timeout — both paths continue macro execution
