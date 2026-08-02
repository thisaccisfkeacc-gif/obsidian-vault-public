---
name: wpf-patterns
description: WPF MVVM patterns, data binding, threading, and UI patterns specific to PowerX Keys. Use when working on Views (XAML), ViewModels, INotifyPropertyChanged, Dispatcher threading, or WPF/WinForms hybrid issues.
tags: [skill, wpf, mvvm, csharp, ui, threading]
date: 2026-06-08
status: active
sources:
  - wiki/architecture/component-relationships.md
  - wiki/architecture/overview.md
---

# 🖼️ Skill: WPF Patterns for PowerX Keys

PowerX Keys uses WPF + WinForms hybrid with strict MVVM. This skill covers the patterns, pitfalls, and conventions specific to this codebase.

## MVVM Structure

```
MainWindow (View)
    └── MainViewModel (root VM — controls page navigation)
         ├── ScriptLibraryViewModel → ScriptLibraryView.xaml
         ├── MacroEditorViewModel  → MacroEditorView.xaml
         ├── SettingsDashboardViewModel → SettingsDashboardView.xaml
         └── AIAssistantViewModel → AIAssistantView.xaml
```

**Navigation pattern:** ViewModel-first, not view-first.  
To navigate: set `MainViewModel.CurrentPage` to the desired ViewModel.

## INotifyPropertyChanged Pattern

All ViewModels inherit from a base class with `OnPropertyChanged()`. Use:

```csharp
private string _myProperty;
public string MyProperty
{
    get => _myProperty;
    set
    {
        if (_myProperty == value) return;
        _myProperty = value;
        OnPropertyChanged();
        // If other properties depend on this:
        OnPropertyChanged(nameof(DerivedProperty));
    }
}
```

**Common mistake:** Forgetting to call `OnPropertyChanged()` → UI never updates.

## Commands (ICommand Pattern)

PowerX Keys uses `RelayCommand` (or `DelegateCommand`). Pattern:

```csharp
// In ViewModel constructor or property:
public ICommand MyCommand { get; }

// Init:
MyCommand = new RelayCommand(ExecuteMyCommand, CanExecuteMyCommand);

private void ExecuteMyCommand()
{
    // Do the action
}

private bool CanExecuteMyCommand() => _someCondition;
```

**Binding in XAML:**
```xml
<Button Command="{Binding MyCommand}" Content="Click Me" />
```

## Threading Rules (Critical)

WPF UI must be updated on the UI thread. This is the most common source of crashes.

```csharp
// ✅ Safe — already on UI thread (event handler, command)
MyProperty = "value";

// ✅ Safe — dispatching from background thread
Application.Current.Dispatcher.Invoke(() =>
{
    MyProperty = "value";
});

// ✅ Async-safe
await Application.Current.Dispatcher.InvokeAsync(() =>
{
    MyProperty = "value";
});

// ❌ Crash — updating UI from Task.Run or background thread without Dispatcher
Task.Run(() => { MyProperty = "value"; }); // WRONG
```

**Rule:** Any UI update from `async void`, `Task.Run`, or event handlers from non-UI sources → use `Dispatcher.Invoke`.

## ObservableCollection

For lists bound to UI, always use `ObservableCollection<T>`:

```csharp
public ObservableCollection<MacroItem> Macros { get; } = new();

// Modifying from background thread → must use Dispatcher
Application.Current.Dispatcher.Invoke(() =>
{
    Macros.Add(newMacro);
    Macros.Remove(oldMacro);
});
```

## Data Binding in XAML

```xml
<!-- Bind to ViewModel property -->
<TextBlock Text="{Binding MacroName}" />

<!-- Two-way binding (input) -->
<TextBox Text="{Binding SearchText, UpdateSourceTrigger=PropertyChanged}" />

<!-- Bind to nested property -->
<TextBlock Text="{Binding SelectedMacro.Name}" />

<!-- Bind with converter -->
<TextBlock Text="{Binding Status, Converter={StaticResource StatusToColorConverter}}" />

<!-- Bind visibility to bool -->
<Grid Visibility="{Binding IsLoading, Converter={StaticResource BoolToVisibilityConverter}}" />
```

## MacroEditorViewModel — Partial Class Map

When editing `MacroEditorViewModel`, open the RIGHT partial file:

| File | Contains |
|------|---------|
| `MacroEditorViewModel.Core.cs` | Constructor, initialization, core state |
| `MacroEditorViewModel.Properties.cs` | All public properties and backing fields |
| `MacroEditorViewModel.Commands.cs` | All `ICommand` implementations |
| `MacroEditorViewModel.Recording.cs` | Macro recording logic |
| `MacroEditorViewModel.Capture.cs` | UI element and image capture |
| `MacroEditorViewModel.Optimization.cs` | Performance optimization features |

## WPF/WinForms Hybrid Gotchas

PowerX Keys uses both WPF and WinForms — this causes type conflicts:

```csharp
// ⚠️ Conflict — both namespaces have keys/controls
// Be explicit:
using WinFormsKeys = System.Windows.Forms.Keys;
using WpfKey = System.Windows.Input.Key;
```

See [[win32-keyboard-gotchas]] for full list of keyboard detection issues.

## DependencyProperty (Custom Controls Only)

Only use `DependencyProperty` for custom WPF controls or attached behaviors. For normal ViewModel → View data flow, use standard `INotifyPropertyChanged`.

```csharp
// Only for custom controls
public static readonly DependencyProperty MyDPProperty =
    DependencyProperty.Register(nameof(MyDP), typeof(string), typeof(MyControl));

public string MyDP
{
    get => (string)GetValue(MyDPProperty);
    set => SetValue(MyDPProperty, value);
}
```

## Common Bugs to Avoid

| Bug | Cause | Fix |
|-----|-------|-----|
| UI freezes on load | Heavy operation on UI thread | Move to `Task.Run`, update back via `Dispatcher` |
| List doesn't update | Using `List<T>` instead of `ObservableCollection<T>` | Switch to `ObservableCollection<T>` |
| UI not refreshing | Missing `OnPropertyChanged()` | Add the call in the setter |
| Cross-thread exception | UI updated from background thread | Wrap in `Dispatcher.Invoke` |
| Command not firing | `CanExecute` returning false | Check the condition in `CanExecuteMyCommand()` |
| Type conflict | WPF + WinForms both imported | Use fully qualified names |

## Related Pages

- [[component-relationships]] — Full component map
- [[macro-editor]] — MacroEditorViewModel deep dive
- [[script-library]] — ScriptLibraryViewModel overview
- [[win32-keyboard-gotchas]] — Critical Win32 + WPF keyboard issues
