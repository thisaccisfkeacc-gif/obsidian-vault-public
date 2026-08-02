# Block Audit: Notification (Deep — Cross-Path)

**Date:** 2026-07-24
**Scope:** C# in-process, AHK compiled, AHK preview (SingleStep)

---

## BUGS

### 1. C# in-process ignores NotificationSilent and SoundType
- **Severity:** Medium
- **Location:** MacroExecutionService.cs:1622-1657
- **Problem:** C# in-process execution completely ignores `NotificationSilent` flag and `SoundType`. AHK compiled path correctly handles both (adds `+16` for silent, plays SoundPlay sounds). C# preview also handles sounds.

### 2. NotifyIcon not disposed if DispatcherTimer fails
- **Severity:** Low
- **Location:** MacroExecutionService.cs:1632
- **Problem:** Creates `new NotifyIcon()` but no `finally` block. If macro cancelled mid-execution, NotifyIcon stays visible in system tray.

### 3. SoundType case mismatch
- **Severity:** Low
- **Location:** MacroEditorViewModel.Commands.cs:504
- **Problem:** Preview notification sound cases use PascalCase (`"Success"`, `"Error"`). SystemSound block uses lowercase (`"success"`, `"error"`). A notification with `SoundType = "success"` falls to default instead of playing tada.wav.

---

## INCONSISTENCIES

1. Sound paths: Notification AHK hardcodes `C:\Windows\Media\tada.wav`. SystemSound AHK uses same paths. C# SystemSound uses bundled `Sounds/` directory. Three different sound sources.
2. C# in-process ignores `NotificationSilent`. AHK respects it.
3. C# in-process doesn't play notification sounds. C# Preview does. AHK does.

---

## VERIFIED OK

- AHK TrayTip syntax correct (function call with parentheses)
- Icon bit flags correct (Info=1, Warning=2, Error=3, Silent=+16)
- Auto-close via SetTimer correct
