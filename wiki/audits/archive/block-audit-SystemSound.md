# Block Audit: SystemSound (Deep — Cross-Path)

**Date:** 2026-07-24
**Scope:** C# in-process, AHK compiled, AHK preview (SingleStep)

---

## BUGS

### 1. SoundPlayer not disposed in C#
- **Severity:** Low
- **Location:** MacroExecutionService.cs:1861
- **Problem:** `new System.Media.SoundPlayer(step.SoundFilePath)` not wrapped in `using`. Resource leak on each play.

### 2. Sound type names differ between paths
- **Severity:** Medium
- **Location:** ScriptCompilerService.cs vs MacroExecutionService.cs
- **Problem:** AHK has title-case names: `"Success Chime"`, `"Error Chord"`. C# has lowercase: `"success"`, `"error"`. User sees different options depending on execution path.

### 3. Default fallback differs between paths
- **Severity:** Low
- **Location:** Multiple
- **Problem:** C# defaults to `Console.Beep(800, 150)`. AHK defaults to `SoundPlay("*-1", 1)` (system asterisk). C# Preview defaults to bundled `notification.wav`. Three different defaults.

---

## INCONSISTENCIES

1. C# bundled sounds: `{AppDir}/Sounds/{name}.wav`. AHK: `C:\Windows\Media\{name}.wav`. Different paths.
2. Sound type naming: three different schemes (lowercase, title-case, PascalCase).

---

## VERIFIED OK

- AHK `FileExist()` guard for Custom File — correct
- AHK `try/catch` around all sound calls — correct
- AHK `SoundPlay(path, 1)` flag=1 means wait — correct
- C# `PlayBundledSound` has try/catch with Console.Beep fallback — correct
