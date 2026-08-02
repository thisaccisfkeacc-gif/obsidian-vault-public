---
tags: [fix, ux, phase-2]
date: 2026-07-23
status: done
---

# 🔧 Fix 11: Settings Persistence

**Priority:** 🟡 Medium
**Effort:** 1 hour
**Risk:** 🟢 Low

---

## Problem

User settings are not properly saved/loaded, causing:
1. Settings lost on app restart
2. Window position not remembered
3. Theme preference reset
4. Hotkey assignments lost

## Current Issues

### Issue 1: No Save on Change
```csharp
// Settings change but not auto-saved
Settings.Default.Theme = "Dark";
// Missing: Settings.Default.Save();
```

### Issue 2: No Load on Startup
```csharp
// App starts with defaults, not saved values
var theme = "Light"; // Should load from Settings
```

### Issue 3: Window State Not Saved
```csharp
// Window position hardcoded
Left = 100;
Top = 100;
// Should save/restore from Settings
```

## Proposed Fix

### Part A: Auto-save on change
```csharp
public string Theme
{
    get => _theme;
    set
    {
        _theme = value;
        Settings.Default.Theme = value;
        Settings.Default.Save(); // Auto-save
        OnPropertyChanged();
    }
}
```

### Part B: Load on startup
```csharp
// In App.xaml.cs
protected override void OnStartup(StartupEventArgs e)
{
    Settings.Default.Reload();
    base.OnStartup(e);
}
```

### Part C: Save/restore window state
```csharp
// In MainWindow.xaml.cs
protected override void OnClosed(EventArgs e)
{
    Settings.Default.WindowLeft = Left;
    Settings.Default.WindowTop = Top;
    Settings.Default.WindowWidth = Width;
    Settings.Default.WindowHeight = Height;
    Settings.Default.Save();
    base.OnClosed(e);
}
```

## Settings to Persist

1. Theme preference (Dark/Light)
2. Window position and size
3. Hotkey assignments
4. Recent files list
5. Recording preferences
6. Soundboard settings

## Expected Impact

- **Settings survive restart** — users don't lose configuration
- **Window remembers position** — better UX
- **Hotkeys persist** — no re-configuration needed

---

**Awaiting Review:** Other agent to confirm "Agree" or provide counterargument.
