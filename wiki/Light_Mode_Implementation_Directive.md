# Light Mode Implementation Directive

This document provides precise instructions for the development agent to implement visual and contrast fixes across the application for Light Mode.

---

## 1. App.xaml (Global ComboBox Styles)
Locate the style `<Style x:Key="PremiumComboBoxStyle" TargetType="ComboBox">` (around line 972) and make the following replacements:

1.  **Nested TextBlock Style:**
    *   Find:
        ```xml
        <Style TargetType="TextBlock">
            <Setter Property="TextTrimming" Value="CharacterEllipsis"/>
            <Setter Property="Foreground" Value="White"/>
        ```
    *   Change to:
        ```xml
        <Style TargetType="TextBlock">
            <Setter Property="TextTrimming" Value="CharacterEllipsis"/>
            <Setter Property="Foreground" Value="{DynamicResource TokenTextPrimaryBrush}"/>
        ```

2.  **ComboBox Default Setters:**
    *   Find:
        ```xml
        <Setter Property="Background" Value="#2A2B33"/>
        <Setter Property="Foreground" Value="#FFFFFF"/>
        <Setter Property="BorderBrush" Value="#31333C"/>
        ```
    *   Change to:
        ```xml
        <Setter Property="Background" Value="{DynamicResource TokenInputBgBrush}"/>
        <Setter Property="Foreground" Value="{DynamicResource TokenTextPrimaryBrush}"/>
        <Setter Property="BorderBrush" Value="{DynamicResource TokenBorderDefaultBrush}"/>
        ```

3.  **Dropdown Popup Border Background & BorderBrush:**
    *   Find (around line 1032):
        ```xml
        <Border x:Name="DropDownBorder" Background="#1C1D21" BorderThickness="1" BorderBrush="#353742" Margin="0,4,0,0" CornerRadius="6">
        ```
    *   Change to:
        ```xml
        <Border x:Name="DropDownBorder" Background="{DynamicResource TokenPanelBgBrush}" BorderThickness="1" BorderBrush="{DynamicResource TokenBorderDefaultBrush}" Margin="0,4,0,0" CornerRadius="6">
        ```

---

## 2. Views/MacroStepCard.xaml (Drag Handle & Delete Buttons)
Locate the control layout in `MacroStepCard.xaml` and resolve low-contrast elements:
*   Find the three-line drag handle icon/path (often displaying `::` or vertical grip dots) and set its Foreground or opacity to `{DynamicResource TokenTextMutedBrush}` instead of hardcoded white.
*   Find the delete `x` button/icon on the right side of the card and change its Foreground to `{DynamicResource TokenTextMutedBrush}`.

---

## 3. Views/MacroEditorView.xaml (Editor Toolbar Icons)
Locate the buttons representing `Undo`, `Redo`, `Filter`, and `Timer` in the editor toolbar:
*   Find the nested icon `TextBlock`s inside the buttons and change their Foreground properties to `{DynamicResource TokenTextSecondaryBrush}` so they are clearly visible against the light button backgrounds.

---

## 4. Views/TextSnippetsView.xaml (Snippet Gear Icon)
*   Locate the settings gear button icon inside each text snippet card.
*   Update the `TextBlock` rendering the gear icon (`&#xE713;`) to use `Foreground="{DynamicResource TokenTextMutedBrush}"` instead of hardcoded white.

---

## 5. Views/SettingsDashboardView.xaml (UI Zoom & Keyboard Manager)

1.  **Zoom Preset Buttons:**
    *   Find the unselected zoom buttons (e.g. `80%`, `90%`).
    *   Update their `Foreground` setter from `#FFFFFF` to `{DynamicResource TokenTextSecondaryBrush}`.

2.  **Keyboard Layout Manager Key Style:**
    *   Find the key border/text label template styles inside the Keyboard settings tab.
    *   Ensure default key labels use `Foreground="{DynamicResource TokenTextPrimaryBrush}"` instead of white.
    *   Ensure modifier keys (SHIFT, CTRL, ALT, WIN) use highly legible combinations in Light Mode (e.g., light-grey background `{DynamicResource TokenMutedBgBrush}` with dark text `{DynamicResource TokenTextPrimaryBrush}`).

---

## 6. Views/DropdownPromptWindow.xaml (Prompt Dialog ComboBox)
*   Locate the ComboBox named `OptionsComboBox` (around line 113).
*   Change the hardcoded `Foreground="White"` property to `Foreground="{DynamicResource TokenTextPrimaryBrush}"` so the selected option text is legible against the light input background in Light Mode.
