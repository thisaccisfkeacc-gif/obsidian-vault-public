---
tags: [feature, ahk, service]
date: 2026-08-01
sources:
  - C:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\PowerX.Core\Models\AppConfig.cs
  - C:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\PowerX_Keys_V2\App.xaml.cs
  - C:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\PowerX.Services\Services\ScriptCompilerService.cs
  - C:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\PowerX.UI\Views\CustomActionCard.xaml
  - C:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\PowerX.UI\Converters\TriggerModeToReadableConverter.cs
status: completed
---

# Schedule Trigger (Time & Day Scheduling) 🕒

**Summary:** The Schedule trigger allows macros to be executed automatically based on time intervals, daily times, or weekly days. Macros run via AHK polling (interval timers + time-of-day checks) while the app/engine is running.

## 🏗️ Architecture & Modes

When a user configures a macro to trigger via **Schedule** (previously *Auto-Repeat*), they select from three sub-modes:

| Sub-Mode | Description | AHK Implementation |
|----------|-------------|--------------------|
| **Interval** | Repeats every X seconds (e.g., 10s, 1m, 1h). | AHK `SetTimer(func, intervalMs)` loop. |
| **Daily** | Runs at a specific time (e.g., 9:00 AM) every day. | AHK checks `A_Now` time string every 30s. |
| **Weekly** | Runs at a specific time on chosen days of the week. | AHK checks time + `A_WDay` every 30s. |

### Double-Fire Prevention
In Daily and Weekly modes, AHK guards against multiple executions within the same day by storing the date of the last successful execution in a `{funcName}_LastFired` variable (initialized to `""` and set to the current `YYYYMMDD` after firing).

### Startup Bypass
If the macro has `ScheduleRunOnStart` enabled, AHK invokes the generated function directly on engine startup — the timer fires once after 1 second (`SetTimer(() => funcName(), -1000)`). For interval schedules without `ScheduleRunOnStart`, an auto-buffer run happens once after 10 seconds. There is no `force` parameter in the generated functions.

---

## 🔌 Windows Task Scheduler Integration (Beta)

> ⚠️ **Removed.** The Windows Task Scheduler backend (`ScheduleTaskService.cs`, `schtasks.exe` / `PowerX_Schedule_{Guid}` task creation, and the `SyncAllTasks` sync flow) has been removed from the app. The config marker remains at `PowerX.Core\Models\AppConfig.cs:1172` ("Task Scheduler Backend removed - archived in Obsidian"), and the "Run even when app is closed" behavior no longer exists. Schedule triggers now run purely via the AHK polling approach described above — they only fire while the app/engine is running. (Archived in Obsidian if details are needed.)

---

## 📁 Key Files

- [AppConfig.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Core/Models/AppConfig.cs) — ActionItem data model properties (incl. removed-backend marker).
- [App.xaml.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/App.xaml.cs) — App startup wiring.
- [ScriptCompilerService.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Services/Services/ScriptCompilerService.cs) — Compiles the AHK script with interval timers and time/day-of-week polling.
- [CustomActionCard.xaml](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/Views/CustomActionCard.xaml) — UI dropdowns, hour/minute pickers, day toggles.
- [TriggerModeToReadableConverter.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/Converters/TriggerModeToReadableConverter.cs) — UI label formatting mapping.

## Related Pages

- [[script-library]]
- [[planned-features]]
- [[execution-pipeline]]
