# 🔍 Deep Audit Prompt 3: Macro Editor Navigation, Context Menus & Recording Scan

You are tasked with executing a deep structural and interactive audit of the **PowerX Keys** Macro Editor UI, toolbar actions, context menus, step reordering, and live recording.

---

## 💡 Research Strategy
- **Search Obsidian Vault First**: Search `c:\Users\Maaz\Documents\New folder\Obsidian Vault\` for existing documentation, GOTCHAS, and schemas before doing full codebase searches.
- **Use Graphify Knowledge Graph**: Check `graphify-out/` or use `graphify query` for fast node/relationship lookup across components.

---

## 🎯 Target Scope
- **Root Directory**: `c:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\`
- **Core Files**: `PowerX.UI\Views\MacroEditorView.xaml`, `PowerX.UI\Views\MacroEditorView.xaml.cs`, `PowerX.UI\ViewModels\MacroEditorViewModel.cs`, `PowerX.UI\ViewModels\MacroEditorViewModel.Commands.cs`, `PowerX.UI\ViewModels\MacroEditorViewModel.Capture.cs`

---

## 📋 Comprehensive Audit Checklist

### 1. Macro Editor Layout & Navigation
- Audit Macro Editor UI, header toolbar, empty timeline state, and Undo/Redo manager (`Ctrl+Z`, `Ctrl+Y`).
- Verify context-sensitive enablement logic for the **Preview Button** and **Save Button** across empty, dirty, and running states.

### 2. Context Menu & Step Card Actions
- Audit every option in the step card Context Menu:
  - **Preview Step**, **Rename**, **Duplicate**, **Disable/Enable**
  - **Edit Coordinates**, **Move Up**, **Move Down**, **Move into Container**
  - **Delete Step**
- Audit step card left-side drag handle reordering (`DragHandle`) and container nesting/unnesting visuals.
- Audit step card Gear Menus for option popups and parameter bindings.

### 3. Recording Engine Scan
- Audit live recording trigger (`ToggleRecordCommand`), mouse path tracking, keypress capture, and smart delay merging.

---

## 📝 Output Requirement
Please write your detailed findings, verified root causes, line numbers, and actionable bug reports directly to:
`c:\Users\Maaz\Documents\New folder\audit_report_3.md`
