---
tags: [guide, development, feature, workflow]
date: 2026-08-01
sources:
  - Models/
  - ViewModels/
  - Services/ScriptCompilerService.cs
status: current
---

# Adding a Feature 🛠️

Step-by-step guide for adding a new feature to PowerX Keys. Follow the MVVM pattern consistently.

## Decision: Does it affect macro execution?

This determines how many layers you need to touch:

| Feature Type | Layers to Modify |
|-------------|-----------------|
| **UI-only** (new setting toggle) | Model → View → ViewModel |
| **New macro step type** | Model → View → ViewModel → Compiler |
| **New trigger mode** | Model → View → ViewModel → Compiler |
| **New service** (background feature) | Service → Manager → ViewModel → View |

## Step-by-Step: Adding a New Macro Step Type

### 1. Add to the Model

In [MacroItem.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Core/Models/MacroItem.cs):

- Add a new value to the `MacroStepType` enum
- Add any new properties to `MacroStep` (with sensible defaults)
- Ensure JSON serialization works (properties auto-serialize)

### 2. Add UI in the View

In [MacroStepCard.xaml](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/Views/MacroStepCard.xaml):

- Add a new `DataTrigger` or section for the step type
- Follow the "Clean UI Rule": hide child controls unless their parent setting is active
- Use Segoe MDL2 Assets for icons (match existing aesthetic)

### 3. Wire Logic in the ViewModel

In `MacroEditorViewModel` (the appropriate partial class file):

- Add the step type to the "Add Step" dropdown in `Commands.cs`
- Handle any special recording behavior in `Recording.cs`
- Add validation rules (check for required fields)

### 4. Update the Compiler

In [ScriptCompilerService.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Services/Services/ScriptCompilerService.cs):

- Add a `case` for the new `MacroStepType` in the main compilation switch
- Generate valid AHK v2 syntax
- Handle disabled steps (`IsDisabled == true` → skip)
- Handle nested children if it's a logic block

### 5. Test

- Build the project (`dotnet build`)
- Create a macro with the new step type
- Run it and check the generated `.ahk` file in `%DOCUMENTS%/PowerX_Keys/Engine/`
- Verify the step works with all trigger modes

## Step-by-Step: Adding a New Setting

### 1. Model
Add property to `SettingsModel` in [AppConfig.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Core/Models/AppConfig.cs) with a default value.

### 2. ViewModel
Add a bound property in `SettingsDashboardViewModel` that reads/writes `ConfigManager.Current.Settings.YourSetting`.

### 3. View
Add a toggle/dropdown/slider in `SettingsDashboardView.xaml` bound to the ViewModel property.

### 4. Factory Reset
Add the property to `NotifyAllSettings()` so it syncs on reset.

### 5. Compiler (if needed)
If the setting affects execution, read it in `ScriptCompilerService.CompileMasterScript()`.

## Checklist ✅

- [ ] New properties have default values (existing configs won't break)
- [ ] Database schema migration handled (if adding SQLite columns)
- [ ] `NotifyAllSettings()` includes new setting (for Factory Reset)
- [ ] No `ExitApp` in AHK output (use `return` flow)
- [ ] Disabled steps are skipped in compilation
- [ ] Undo/Redo snapshot pushed before mutations
- [ ] Navigation guard (`IsDirty`) triggered on changes

## Key Files

- [MacroItem.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Core/Models/MacroItem.cs)
- [AppConfig.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Core/Models/AppConfig.cs)
- [ScriptCompilerService.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.Services/Services/ScriptCompilerService.cs)
- [MacroStepCard.xaml](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX.UI/Views/MacroStepCard.xaml)

## Related Pages

- [[agent-onboarding]]
- [[macro-item]]
- [[app-config]]
- [[execution-pipeline]]
- [[macro-editor]]
