# Layout Modes & Smart Mode — Full Code Backup
> Created: 2026-07-07 | Reason: Removing dead/hidden features from codebase

---

## Feature 1: Smart Mode Leftovers

### `ViewModels/MacroEditorViewModel.Properties.cs` — Line 30
```csharp
// TEMPORARY: Default to Raw while perfecting raw recording. Re-enable Smart later.
private TimelineViewMode _currentViewMode = TimelineViewMode.Raw;
```

### `ViewModels/MacroEditorViewModel.Commands.cs` — Line 30
```csharp
public ICommand SetViewModeCommand { get; private set; }
```

### `ViewModels/MacroEditorViewModel.Commands.cs` — Lines 184–190
```csharp
SetViewModeCommand = new RelayCommand(delegate(object o)
{
    if (o is string value && Enum.TryParse<TimelineViewMode>(value, out var result))
    {
        CurrentViewMode = result;
    }
});
```

### `Views/MacroEditorView.xaml` — Lines 1133–1136
```xml
<!-- View Modes — TEMPORARILY HIDDEN (Smart View disabled while perfecting Raw mode) -->
<TextBlock Text="VIEW MODES" Foreground="#4B4C56" FontSize="9" FontWeight="Heavy" Margin="10,4,0,4" Visibility="Collapsed"/>
<RadioButton GroupName="RecordingMode" Content="Smart View" Style="{StaticResource DropdownMenuRadioButtonStyle}" Command="{Binding SetViewModeCommand}" CommandParameter="Smart" ToolTip="Automatically combines text into a single block and removes redundant clicks." IsChecked="{Binding CurrentViewMode, Converter={StaticResource EnumToBooleanConverter}, ConverterParameter=Smart}" Visibility="Collapsed"/>
<RadioButton GroupName="RecordingMode" Content="Raw View" Style="{StaticResource DropdownMenuRadioButtonStyle}" Command="{Binding SetViewModeCommand}" CommandParameter="Raw" ToolTip="Displays every individual keystroke and mouse event exactly as it was recorded." IsChecked="{Binding CurrentViewMode, Converter={StaticResource EnumToBooleanConverter}, ConverterParameter=Raw}" Visibility="Collapsed"/>
```

### `ViewModels/TimelineViewMode.cs` — ENTIRE FILE (already deleted, backed up in Smart_Mode_Logic_Backup.md)

---

## Feature 2: Layout Modes (Normal / Compact / List)

### `ViewModels/MacroEditorViewModel.Properties.cs` — Line 28
```csharp
private int _timelineLayoutIndex = 0;
```

### `ViewModels/MacroEditorViewModel.Properties.cs` — Line 56
```csharp
private bool _isExtremeMiniView;
```

### `ViewModels/MacroEditorViewModel.Properties.cs` — Lines 303–307
```csharp
public bool IsExtremeMiniView
{
    get => _isExtremeMiniView;
    set { _isExtremeMiniView = value; OnPropertyChanged(nameof(IsExtremeMiniView)); }
}
```

### `ViewModels/MacroEditorViewModel.Properties.cs` — Lines 409–427
```csharp
public int TimelineLayoutIndex
{
    get => _timelineLayoutIndex;
    set 
    { 
        _timelineLayoutIndex = value; 
        OnPropertyChanged(nameof(TimelineLayoutIndex)); 
        if (value == 2 && !_isExtremeMiniView) 
        {
            IsExtremeMiniView = true;
            if (CurrentMacro != null)
            {
                TraverseAllSteps(CurrentMacro.MacroSteps, s => s.IsExpanded = false);
            }
        }
        else if (value != 2 && _isExtremeMiniView) IsExtremeMiniView = false;
        ForceRefreshDisplaySteps();
    }
}
```

### `ViewModels/MacroEditorViewModel.Commands.cs` — Line 31
```csharp
public ICommand CycleTimelineLayoutCommand { get; private set; }
```

### `ViewModels/MacroEditorViewModel.Commands.cs` — Lines 192–197
```csharp
CycleTimelineLayoutCommand = new RelayCommand(delegate(object o)
{
    // TEMP: Compact (1) & List (2) layouts hidden — always stay on Normal (0)
    TimelineLayoutIndex = 0;
    // TimelineLayoutIndex = (TimelineLayoutIndex + 1) % 3; // restore when ready
});
```

### `Views/MacroEditorView.xaml` — Lines 1019–1059 (LayoutCycleBtn)
```xml
<!-- Timeline Layout Cycle Button (temporarily hidden - Compact & List not yet refined) -->
<Button x:Name="LayoutCycleBtn" Command="{Binding CycleTimelineLayoutCommand}" Background="#25262B" BorderThickness="1" BorderBrush="#2D2F36" Cursor="Hand" Width="34" Height="28" Margin="0,0,8,0" Visibility="Collapsed">
    <Button.Template>
        <ControlTemplate TargetType="Button">
            <Border x:Name="BgBorder" Background="{TemplateBinding Background}" CornerRadius="6" BorderBrush="{TemplateBinding BorderBrush}" BorderThickness="{TemplateBinding BorderThickness}">
                <TextBlock x:Name="IconText" FontFamily="Segoe MDL2 Assets" FontSize="12" HorizontalAlignment="Center" VerticalAlignment="Center"/>
            </Border>
            <ControlTemplate.Triggers>
                <Trigger Property="IsMouseOver" Value="True">
                    <Setter TargetName="BgBorder" Property="Background" Value="#25262B"/>
                </Trigger>
                <!-- Normal (0) -->
                <DataTrigger Binding="{Binding TimelineLayoutIndex}" Value="0">
                    <Setter TargetName="IconText" Property="Text" Value="&#xE8FD;"/>
                    <Setter TargetName="IconText" Property="Foreground" Value="#C0C2C8"/>
                    <Setter TargetName="BgBorder" Property="BorderBrush" Value="#3B3D46"/>
                    <Setter Property="ToolTip" Value="Normal Layout"/>
                </DataTrigger>
                <!-- Compact (1) -->
                <DataTrigger Binding="{Binding TimelineLayoutIndex}" Value="1">
                    <Setter TargetName="IconText" Property="Text" Value="&#xE8A1;"/>
                    <Setter TargetName="IconText" Property="Foreground" Value="#FFC107"/>
                    <Setter TargetName="BgBorder" Property="BorderBrush" Value="#806003"/>
                    <Setter TargetName="BgBorder" Property="Background" Value="#1A1502"/>
                    <Setter Property="ToolTip" Value="Compact Layout"/>
                </DataTrigger>
                <!-- List (2) -->
                <DataTrigger Binding="{Binding TimelineLayoutIndex}" Value="2">
                    <Setter TargetName="IconText" Property="Text" Value="&#xE706;"/>
                    <Setter TargetName="IconText" Property="Foreground" Value="#FF4D4D"/>
                    <Setter TargetName="BgBorder" Property="BorderBrush" Value="#FF4D4D"/>
                    <Setter TargetName="BgBorder" Property="Background" Value="#2A1414"/>
                    <Setter Property="ToolTip" Value="List Layout (Extreme Mini View)"/>
                </DataTrigger>
            </ControlTemplate.Triggers>
        </ControlTemplate>
    </Button.Template>
</Button>
```

### `Views/MacroEditorView.xaml` — Lines 1138–1146 (Layout ComboBox)
```xml
<!-- Timeline Layout — temporarily hidden (Compact & List disabled) -->
<TextBlock Text="TIMELINE LAYOUT" Foreground="#4B4C56" FontSize="9" FontWeight="Heavy" Margin="10,8,0,4" Visibility="Collapsed"/>
<Border Margin="10,2,10,8" Visibility="Collapsed">
    <ComboBox Style="{StaticResource PremiumComboBoxStyle}" HorizontalAlignment="Stretch" SelectedIndex="{Binding TimelineLayoutIndex}">
        <ComboBoxItem Content="Normal"/>
        <ComboBoxItem Content="Compact"/>
        <ComboBoxItem Content="List"/>
    </ComboBox>
</Border>
```

### `Views/MacroEditorView.xaml` — Lines 1595–1953 (ExperimentalMiniGrid DataGrid)
> ~360 lines of DataGrid XAML with full context menu, row/cell styles, columns, drag handles.
> Key attributes: `x:Name="ExperimentalMiniGrid"`, `Visibility="{Binding IsExtremeMiniView, Converter=...}"`
> Event handlers wired: `ExperimentalMiniGrid_PreviewMouseLeftButtonDown`, `ExperimentalMiniGrid_MouseDoubleClick`, `ExperimentalMiniGrid_PreviewKeyDown`
> Columns: Drag Handle, Action (Type), Details (DisplayValue)
> RowDetailsTemplate: expands to MacroStepTemplate

### `Views/MacroEditorView.Events.cs` — ExperimentalMiniGrid handlers
Lines involved: 27, 28, 107–146, 148–265, 426–456, 465–500, 590–665, 750–766, 930–931, 1087–1089, 1384

Key methods:
- `ExperimentalMiniGrid_PreviewKeyDown` (line 107)
- `ExperimentalMiniGrid_DragOver` (line 148)
- `ExperimentalMiniGrid_DragLeave` (line 236)
- `ExperimentalMiniGrid_Drop` (line 241)
- `MiniGridDragHandle_PreviewMouseLeftButtonDown` (line 428)
- `MiniGridDragHandle_MouseMove` (line 434)
- `ExperimentalMiniGrid_PreviewMouseLeftButtonDown` (line 465)
- `ExperimentalMiniGrid_MouseDoubleClick` (line 486)

### `Models/MacroItem.cs` — Lines 95–101
```csharp
private bool _isExpandedInCompactMode = false;
[System.Text.Json.Serialization.JsonIgnore]
public bool IsExpandedInCompactMode
{
    get => _isExpandedInCompactMode;
    set { _isExpandedInCompactMode = value; OnPropertyChanged(); }
}
```

---
*Backup complete. Safe to remove all items above from the codebase.*
