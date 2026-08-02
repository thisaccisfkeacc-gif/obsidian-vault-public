# Block Audit: Text (Deep — Cross-Path)

**Date:** 2026-07-24
**Scope:** C# in-process, AHK compiled, AHK preview (SingleStep)

---

## Summary

Text block types text character-by-character (slow mode) or uses clipboard fast paste. Supports variable linking (pull from UserVariables map set by UserInput blocks).

---

## BUGS

### 1. AHK fast-paste sleep is 3.3x shorter than C#
- **Severity:** Medium
- **Location:** ScriptCompilerService.cs:2736 vs MacroExecutionService.cs
- **Problem:** AHK restores clipboard after only 150ms. C# waits 500ms. Slow apps (Word, browsers) may not finish reading clipboard in 150ms, causing paste failure in AHK path.

### 2. Variable name sanitization inconsistent
- **Severity:** Medium
- **Location:** MacroExecutionService.cs:1396 vs ScriptCompilerService.cs:2709
- **Problem:** C# strips to `IsLetterOrDigit` only. AHK allows `IsLetterOrDigit || c == '_'`. Variable `My_Var` becomes `MyVar` in C# but stays `My_Var` in AHK — lookup miss.

### 3. Text has NO preview support
- **Severity:** Medium
- **Location:** ScriptCompilerService.SingleStep.cs
- **Problem:** `CompileSingleStepTestScript()` has no `MacroStepType.Text` branch. Clicking "Test" on a Text step generates an empty script.

### 4. C# clipboard backup swallows exceptions silently
- **Severity:** Low
- **Location:** MacroExecutionService.cs:1429
- **Problem:** If `Clipboard.SetText()` fails (locked by another app), the catch block is empty. Paste still runs but clipboard is wiped without restore.

### 5. Sandbox compiler doesn't handle Text
- **Severity:** Low
- **Location:** ScriptCompilerService.cs:4859-4940
- **Problem:** `CompileSandboxStep` (macro-on-hotkey) doesn't handle `MacroStepType.Text`. Text steps are silently dropped in hotkey-triggered macros.

---

## DEAD CODE

1. `Clipboard.Clear()` in fallback restore — intended to restore but actually just clears whatever's on clipboard.

---

## REDUNDANCIES

1. `SetKeyDelay(50, 10)` emitted before every `SendEvent("{Text}")` call — `{Text}` mode ignores `SetKeyDelay` in AHK v2. No effect.
2. `ClipSaved := ClipboardAll()` saved/restored on every fast-paste step. Redundant if multiple consecutive Text blocks.

---

## MISSING FEATURES

| Feature | C# | AHK Full | AHK Preview |
|---|---|---|---|
| Fast paste (Ctrl+V) | ✅ | ✅ | ❌ (no preview) |
| Slow type (char-by-char) | ✅ (true char-by-char) | ❌ (instant with SetKeyDelay) | ❌ |
| Variable linking | ✅ | ✅ | N/A |
| Newline handling | ✅ (sends {ENTER}) | ✅ (automatic) | N/A |

---

## INCONSISTENCIES

1. Fast-paste sleep: C# = 50ms pre + 500ms post + 50ms restore. AHK = 50ms pre + 150ms post. ~3.3x faster in AHK.
2. Slow type: C# sends one char at a time with 15ms delay. AHK sends entire text at once via `SendEvent("{Text}")`. Fundamentally different.
3. Variable names: C# alphanumeric only. AHK alphanumeric + underscore.

---

## VERIFIED OK

- `EscapeStringForAhkLiteral` — correct for backticks, quotes, newlines, tabs
- AHK clipboard backup/restore using `ClipboardAll()` — correct
- AHK `SendEvent("{Text}")` syntax — correct
- UserVariables initialization — correct
- Empty text guard — consistent across paths
