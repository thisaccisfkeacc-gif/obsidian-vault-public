# 📂 Legacy Feature: Master Kill Switch
**Date Archived:** July 6, 2026
**Status:** Disabled (Extracted to Obsidian)
**Original Location:** `ViewModels/SettingsDashboardViewModel.cs` and `Views/SettingsDashboardView.xaml`

---

## 🧠 ViewModel Logic (`SettingsDashboardViewModel.cs`)
This code managed the dropdown for selecting the kill key and the notification toggle.

```csharp
        public string MasterKillSwitchKey
        {
            get => Services.ConfigManager.Current.Settings.MasterKillSwitchKey;
            set
            {
                if (Services.ConfigManager.Current.Settings.MasterKillSwitchKey != value)
                {
                    Services.ConfigManager.Current.Settings.MasterKillSwitchKey = value;
                    Services.ConfigManager.Save();
                    OnPropertyChanged(nameof(MasterKillSwitchKey));
                }
            }
        }

        public System.Collections.Generic.List<string> AvailableKillSwitchKeys { get; } = new System.Collections.Generic.List<string>
        {
            "Double Escape",
            "Shift + Escape",
            "Ctrl + Shift + Escape",
            "Scroll Lock",
            "Pause",
            "Print Screen",
            "F12",
            "F11",
            "Backspace",
            "Delete"
        };

        public bool KillSwitchNotification
        {
            get => Services.ConfigManager.Current.Settings.KillSwitchNotification;
            set
            {
                if (Services.ConfigManager.Current.Settings.KillSwitchNotification != value)
                {
                    Services.ConfigManager.Current.Settings.KillSwitchNotification = value;
                    Services.ConfigManager.Save();
                    OnPropertyChanged(nameof(KillSwitchNotification));
                }
            }
        }
```

## 🖼️ UI Elements (`SettingsDashboardView.xaml`)
The dropdown (ComboBox) and ToggleSwitch were located here.

```xml
<!-- ComboBox for MasterKillSwitchKey -->
<ComboBox ItemsSource="{Binding AvailableKillSwitchKeys}" 
          SelectedItem="{Binding MasterKillSwitchKey}" ... />

<!-- ToggleSwitch for KillSwitchNotification -->
<ToggleButton IsChecked="{Binding KillSwitchNotification}" ... />
```

---
> [!TIP]
> This feature lets users stop all running macros instantly using a hotkey. If you need it back, just restore these properties and the UI elements.
