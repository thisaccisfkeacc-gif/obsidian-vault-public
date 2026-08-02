# 📂 Legacy Feature: Windows Task Scheduler Backend
**Date Archived:** July 6, 2026
**Status:** Disabled & Hidden (Extracted to Obsidian)
**Original Location:** `Services/ScheduleTaskService.cs` and `Models/AppConfig.cs`

---

## 🧠 Service Logic (`Services/ScheduleTaskService.cs`)
This service handled the integration with Windows Task Scheduler for Daily/Weekly triggers.

```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.IO;
using System.Linq;
using PowerX_Keys_V2.Models;

namespace PowerX_Keys_V2.Services
{
    public static class ScheduleTaskService
    {
        private const string TaskPrefix = "PowerX_Schedule_";

        public static bool CreateTask(ActionItem action)
        {
            if (action == null || string.IsNullOrEmpty(action.Id)) return false;
            if (!action.Enabled || action.IsConflicting) return false;
            RemoveTask(action);

            if (action.TriggerMode != TriggerMode.Schedule || action.ScheduleMode == 0 || !action.ScheduleUseTaskScheduler)
            {
                return true;
            }

            var path = Process.GetCurrentProcess().MainModule?.FileName ?? System.Reflection.Assembly.GetExecutingAssembly().Location;
            string taskName = TaskPrefix + action.Id;
            string timeStr = $"{action.ScheduleHour:D2}:{action.ScheduleMinute:D2}";
            string trValue = $"\"{path}\" --run-scheduled {action.Id}";

            string args = "";
            if (action.ScheduleMode == 1) // Daily
            {
                args = $"/create /tn \"{taskName}\" /tr \"{trValue}\" /sc DAILY /st {timeStr} /f";
            }
            else if (action.ScheduleMode == 2) // Weekly
            {
                var daysList = new List<string>();
                if (action.ScheduleMon) daysList.Add("MON");
                if (action.ScheduleTue) daysList.Add("TUE");
                if (action.ScheduleWed) daysList.Add("WED");
                if (action.ScheduleThu) daysList.Add("THU");
                if (action.ScheduleFri) daysList.Add("FRI");
                if (action.ScheduleSat) daysList.Add("SAT");
                if (action.ScheduleSun) daysList.Add("SUN");
                if (daysList.Count == 0) return true;

                string daysStr = string.Join(",", daysList);
                args = $"/create /tn \"{taskName}\" /tr \"{trValue}\" /sc WEEKLY /d {daysStr} /st {timeStr} /f";
            }

            if (string.IsNullOrEmpty(args)) return false;
            return RunSchtasks(args);
        }

        public static bool RemoveTask(ActionItem action)
        {
            if (action == null || string.IsNullOrEmpty(action.Id)) return false;
            string taskName = TaskPrefix + action.Id;
            return RunSchtasks($"/delete /tn \"{taskName}\" /f");
        }

        public static void SyncAllTasks(IEnumerable<ActionItem> allActions)
        {
            try
            {
                var existingTaskIds = GetExistingTaskIds();
                var targetTaskIds = new HashSet<string>();
                foreach (var action in allActions)
                {
                    if (action.TriggerMode == TriggerMode.Schedule && 
                        action.ScheduleMode > 0 && 
                        action.ScheduleUseTaskScheduler && 
                        action.Enabled && 
                        !action.IsConflicting)
                    {
                        targetTaskIds.Add(action.Id);
                        CreateTask(action);
                    }
                }
                foreach (var taskId in existingTaskIds)
                {
                    if (!targetTaskIds.Contains(taskId))
                    {
                        string taskName = TaskPrefix + taskId;
                        RunSchtasks($"/delete /tn \"{taskName}\" /f");
                    }
                }
            }
            catch (Exception) { }
        }

        private static List<string> GetExistingTaskIds()
        {
            var ids = new List<string>();
            try
            {
                var psi = new ProcessStartInfo
                {
                    FileName = "schtasks.exe",
                    Arguments = "/query /fo CSV /nh",
                    UseShellExecute = false,
                    CreateNoWindow = true,
                    RedirectStandardOutput = true
                };

                using (var process = Process.Start(psi))
                {
                    string output = process.StandardOutput.ReadToEnd();
                    process.WaitForExit();
                    var lines = output.Split(new[] { '\r', '\n' }, StringSplitOptions.RemoveEmptyEntries);
                    foreach (var line in lines)
                    {
                        var parts = line.Split(',');
                        if (parts.Length > 0)
                        {
                            string rawName = parts[0].Trim('"').Trim('\\');
                            if (rawName.StartsWith(TaskPrefix))
                            {
                                string id = rawName.Substring(TaskPrefix.Length);
                                ids.Add(id);
                            }
                        }
                    }
                }
            }
            catch { }
            return ids;
        }

        private static bool RunSchtasks(string arguments)
        {
            try
            {
                var psi = new ProcessStartInfo { FileName = "schtasks.exe", Arguments = arguments, UseShellExecute = false, CreateNoWindow = true };
                using (var process = Process.Start(psi))
                {
                    process.WaitForExit();
                    return process.ExitCode == 0;
                }
            }
            catch { return false; }
        }
    }
}
```

## ⚙️ Settings Backend (`Models/AppConfig.cs`)
These properties were used to store the schedule settings.

```csharp
        // --- Schedule Sub-Mode ---
        private int _scheduleMode = 0; // 0 = Interval, 1 = Daily, 2 = Weekly
        public int ScheduleMode
        {
            get => _scheduleMode;
            set { if (_scheduleMode != value) { _scheduleMode = value; PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(ScheduleMode))); } }
        }

        // --- Time-of-Day ---
        private int _scheduleHour = 9;
        public int ScheduleHour
        {
            get => _scheduleHour;
            set { if (_scheduleHour != value) { _scheduleHour = Math.Clamp(value, 0, 23); PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(ScheduleHour))); PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(ScheduleTimeDisplay))); } }
        }

        private int _scheduleMinute = 0;
        public int ScheduleMinute
        {
            get => _scheduleMinute;
            set { if (_scheduleMinute != value) { _scheduleMinute = Math.Clamp(value, 0, 59); PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(ScheduleMinute))); PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(ScheduleTimeDisplay))); } }
        }

        // --- Day-of-Week Flags ---
        private bool _scheduleMon = true;
        public bool ScheduleMon { get => _scheduleMon; set { if (_scheduleMon != value) { _scheduleMon = value; PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(ScheduleMon))); } } }
        private bool _scheduleTue = true;
        public bool ScheduleTue { get => _scheduleTue; set { if (_scheduleTue != value) { _scheduleTue = value; PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(ScheduleTue))); } } }
        private bool _scheduleWed = true;
        public bool ScheduleWed { get => _scheduleWed; set { if (_scheduleWed != value) { _scheduleWed = value; PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(ScheduleWed))); } } }
        private bool _scheduleThu = true;
        public bool ScheduleThu { get => _scheduleThu; set { if (_scheduleThu != value) { _scheduleThu = value; PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(ScheduleThu))); } } }
        private bool _scheduleFri = true;
        public bool ScheduleFri { get => _scheduleFri; set { if (_scheduleFri != value) { _scheduleFri = value; PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(ScheduleFri))); } } }
        private bool _scheduleSat = false;
        public bool ScheduleSat { get => _scheduleSat; set { if (_scheduleSat != value) { _scheduleSat = value; PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(ScheduleSat))); } } }
        private bool _scheduleSun = false;
        public bool ScheduleSun { get => _scheduleSun; set { if (_scheduleSun != value) { _scheduleSun = value; PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(ScheduleSun))); } } }

        [JsonIgnore]
        public string ScheduleTimeDisplay => $"{ScheduleHour:D2}:{ScheduleMinute:D2}";

        // --- Task Scheduler (Beta) ---
        private bool _scheduleUseTaskScheduler = false;
        public bool ScheduleUseTaskScheduler
        {
            get => _scheduleUseTaskScheduler;
            set { if (_scheduleUseTaskScheduler != value) { _scheduleUseTaskScheduler = value; PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(ScheduleUseTaskScheduler))); } }
        }
```

---
> [!IMPORTANT]
> This code is fully functional but was removed because it was hidden from the UI and making the project "heavy."
