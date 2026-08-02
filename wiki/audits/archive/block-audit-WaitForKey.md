# Block Audit: WaitForKey (Deep — Cross-Path)

**Date:** 2026-07-24
**Scope:** C# in-process, AHK compiled, AHK preview (SingleStep)

---

## BUGS

### 1. C# ignores WaitKeyMode (StrictOK/StrictCancel)
- **Severity:** High
- **Location:** MacroExecutionService.cs:1268-1298
- **Problem:** C# always shows OK/Cancel dialog regardless of mode. StrictOK (any activity cancels) and StrictCancel (any activity continues) are not implemented in C#.

### 2. C# ignores WaitMessageType (ToolTip)
- **Severity:** High
- **Location:** MacroExecutionService.cs:1268-1298
- **Problem:** C# always shows modal popup dialog. When set to "ToolTip", AHK shows lightweight tooltip near cursor. C# ignores this.

### 3. C# ignores ShowWaitMessage
- **Severity:** Medium
- **Location:** MacroExecutionService.cs:1268-1298
- **Problem:** C# always shows dialog. In AHK, `ShowWaitMessage=false` suppresses tooltip/popup. C# has no equivalent.

### 4. No preview support
- **Severity:** Medium
- **Location:** ScriptCompilerService.SingleStep.cs
- **Problem:** WaitForKey has no branch in SingleStep. Preview generates empty script.

---

## INCONSISTENCIES

1. WaitKeyMode semantics: AHK StrictOK is extremely sensitive (any mouse jitter cancels). C# just shows OK/Cancel dialog.
2. DisplayValue doesn't reflect StrictOK/StrictCancel actual behavior — always shows both keys.
3. Popup mode key handling: AHK polls `GetKeyState("P")`. C# uses WPF `PreviewKeyDown` event. Different timing.

---

## VERIFIED OK

- AHK ToolTip mode: GUI creation, positioning, cleanup — correct
- AHK Popup mode: GUI creation, drag support, close handlers — correct
- Token check in all AHK loops — correct
- KeyWait after continue prevents key ghosting — correct
- DarkMessageBoxWindow X-button — correctly throws OperationCanceledException
