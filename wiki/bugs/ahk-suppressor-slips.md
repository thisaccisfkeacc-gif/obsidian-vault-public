# AHK Error Suppressor Slip Risks & Solution

During single-step preview testing, if the AutoHotkey engine encounters execution issues (such as invalid parameters or library loading issues), it spawns standard dialog windows. The current suppressor acts as an interception layer:

```csharp
if (className == "AutoHotkey" || className == "#32770")
{
    if (title.Contains("Error") || title.Contains("PowerX_Engine"))
    {
        PostMessage(hwnd, WM_CLOSE, IntPtr.Zero, IntPtr.Zero);
        // Custom popup logic
    }
}
```

> **Note (2026-08-01):** The snippet above reflects the current code — `AhkErrorSuppressor.cs:78` now matches only `"Error"` or `"PowerX_Engine"`. The earlier `title.Contains("test_preview_step.ahk")` check was removed when the temporary preview script name was dropped.

## Potential Slip Risks
- **Title Mismatches**: Any window shown by AHK that does not contain `"Error"` or `"PowerX_Engine"` in the title will slip past the filter.
- **Timing & Class Shifts**: Dialog classes that aren't strictly `AutoHotkey` or `#32770` won't be captured.

## Packaging Stage Fix
Before final deployment/packaging, enhance `AhkErrorSuppressor.cs` to query the owning Process ID of **any** spawned window. If the owning process name matches `PowerX_Engine` or `AutoHotkey`, suppress the window immediately regardless of window title or class.
