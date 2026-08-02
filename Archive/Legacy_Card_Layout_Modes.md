# Legacy Compact & List Card Layout Modes

## Background
The Macro Editor originally included experimental layout view modes for macro step cards (`Compact Mode` and `List Mode`). These view modes allowed hiding step properties inline until double-clicked, or scanning macro steps in a compressed overview format.

These view modes were removed from the user interface.

## Archived XAML Reference (`Views/MacroStepCard.xaml`)

```xml
<!-- Compact Mode (Index 1) — Properties hidden, double-click to expand -->

<!-- List Mode (Index 2) — Overview mode, click to expand -->

<!-- Show ConfigContainer in Compact Mode if user double-clicked to expand -->

<!-- Show ConfigContainer in List Mode if user double-clicked to expand -->

<!-- Show OptionHandle on hover in Compact Mode -->

<!-- Show OptionHandle on hover in List Mode -->
```

## Related View Models
- View mode state properties were previously controlled via `MacroEditorViewModel` card display layout settings.
