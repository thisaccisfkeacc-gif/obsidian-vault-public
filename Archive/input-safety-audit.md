---
tags: [audit, ui, feature]
date: 2026-07-15
sources:
  - PowerX_Keys_V2/Views/
status: completed
---

# Input Safety Audit — MaxLength

**Summary:** Scanned all XAML files for TextBoxes missing `MaxLength`. Fixed 4 files, 5 TextBoxes total.

## Fixed

| File | TextBox | Added |
|------|---------|-------|
| CaptureLibraryWindow.xaml | Search field | `MaxLength="50"` |
| CoordinateEditDialog.xaml | TxtX (coordinate) | `MaxLength="8"` |
| CoordinateEditDialog.xaml | TxtY (coordinate) | `MaxLength="8"` |
| ExportMacroPickerDialog.xaml | SearchBox | `MaxLength="50"` |
| InputPromptWindow.xaml | InputTextBox (user input) | `MaxLength="500"` |

## Already Compliant (No Changes Needed)

| File | TextBoxes | Status |
|------|-----------|--------|
| KeyboardInputTemplates.xaml | 5 | All have MaxLength |
| MiscTemplates.xaml | 7 | All have MaxLength |
| MacroStepCard.xaml | 0 | No TextBoxes |
| MacroEditorView.xaml | 1 | MaxLength="6" |
| SettingsDashboardView.xaml | 1 | MaxLength="4" |
| ScriptLibraryView.xaml | 4 | MaxLength="30"/"260" |
| CustomActionCard.xaml | 5 | MaxLength="4"/"6" |
| CaptureOverlay.xaml | 0 | No TextBoxes |
| ImageStudioWindow.xaml | 0 | No TextBoxes |
| MouseTemplates.xaml | 5 | MaxLength="5"/"260" |
| SearchTemplates.xaml | 6+ | MaxLength="3"/"5"/"1000" |
| AIAssistantView.xaml | 1 | MaxLength="4000" |
| CrashReportWindow.xaml | 2 | 1 read-only, 1 MaxLength="500" |
| TextSnippetsView.xaml | 2 | MaxLength="30"/"15" |

## Limits Applied

- Numeric fields (X, Y, duration, counts): `8`
- Search fields: `50`
- User text input (prompts, messages): `500`
- Short names/titles: `30`
- File paths/URLs: `260`

## Related Pages

- [[macro-editor]]
- [[settings-dashboard]]
