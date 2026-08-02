# Input Safety Audit — MaxLength & Character Limits

Copy-paste this entire prompt into a new session.

---

## Your Job

Scan **every TextBox** in the PowerX Keys app and ensure they all have appropriate `MaxLength` limits. Fix any that are missing.

---

## Rules

- Scan one XAML file at a time
- For each TextBox without `MaxLength`, add a sensible limit based on what it holds:
  - Numeric fields (Duration, X, Y, counts): `MaxLength="8"`
  - Short text (names, titles, variable names): `MaxLength="50"`
  - Medium text (window titles, file paths, URLs): `MaxLength="260"`
  - Long text (Type Text value, popup messages): `MaxLength="2000"`
  - Color hex: `MaxLength="7"`
- Do NOT change TextBoxes that already have `MaxLength` set
- Do NOT change any logic or behavior — only add the `MaxLength` attribute
- After all files are done, write a summary to: `c:\Users\Maaz\Documents\New folder\Obsidian Vault\core\wiki\audits\input-safety-audit.md`

---

## Project Location

```
c:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\PowerX_Keys_V2\
```

## Files to Scan (in this order)

1. `Views/Templates/KeyboardInputTemplates.xaml`
2. `Views/Templates/MiscTemplates.xaml`
3. `Views/MacroStepCard.xaml`
4. `Views/MacroEditorView.xaml`
5. `Views/SettingsDashboardView.xaml`
6. `Views/ScriptLibraryView.xaml`
7. `Views/CustomActionCard.xaml`
8. `Views/CaptureOverlay.xaml`
9. `Views/ImageStudioWindow.xaml`
10. `Views/CaptureLibraryWindow.xaml`
11. Any other `.xaml` files in `Views/` that contain TextBox controls

---

## Read First

- `c:\Users\Maaz\Documents\New folder\Obsidian Vault\core\SOUL.md`
- `c:\Users\Maaz\Documents\New folder\Obsidian Vault\core\GOTCHAS.md`

---

## Communication Rules

- English only
- Don't explain what you're about to do — just do it
- After fixing a file, just say "Done — [filename] — X TextBoxes updated"
- Keep outputs minimal

---

## Start

Begin with file #1. Read it, find TextBoxes without MaxLength, add appropriate limits, move to next file.
