# Block Audit: Popup (Deep — Cross-Path)

**Date:** 2026-07-24
**Scope:** C# in-process, AHK compiled, AHK preview (SingleStep)

---

## BUGS

### 1. Tooltip mode leaks Form handle in C#
- **Severity:** Low
- **Location:** MacroExecutionService.cs:1502-1509
- **Problem:** Creates `new Form()` as tooltip owner but never disposes it. Each preview execution leaks a hidden Form handle.

### 2. Auto-Timeout dialog can't be dismissed externally
- **Severity:** Low
- **Location:** MacroExecutionService.cs:1496
- **Problem:** `window.ShowDialog()` blocks dispatcher thread. If macro is cancelled during Auto-Timeout, dialog remains open. AHK MsgBox with `T` option auto-closes — no issue there.

---

## INCONSISTENCIES

1. Checkpoint: C# uses `DarkMessageBoxWindow` (WPF custom). AHK uses `MsgBox` with `"OC"` flag. Different visual appearance.
2. Auto-Timeout: C# dialog not externally cancellable. AHK MsgBox auto-closes.
3. Tooltip: C# uses `System.Windows.Forms.ToolTip`. AHK uses `ToolTip()` + `SetTimer`.

---

## VERIFIED OK

- Checkpoint correctly stops macro on Cancel — both paths
- Tooltip AHK path correct
- PopupTimeout default of 3 seconds consistent
- Escaping handles backticks, quotes, newlines
