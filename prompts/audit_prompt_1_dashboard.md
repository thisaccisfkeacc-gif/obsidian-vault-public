# 🔍 Deep Audit Prompt 1: Comprehensive Dashboard & Profile Lifecycle Scan

You are tasked with executing a deep structural and functional audit of the **PowerX Keys** Dashboard and Profile Management system.

---

## 💡 Research Strategy
- **Search Obsidian Vault First**: Search `c:\Users\Maaz\Documents\New folder\Obsidian Vault\` for existing documentation, GOTCHAS, and schemas before doing full codebase searches.
- **Use Graphify Knowledge Graph**: Check `graphify-out/` or use `graphify query` for fast node/relationship lookup across components.

---

## 🎯 Target Scope
- **Root Directory**: `c:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\`
- **Core Files**: `PowerX_Keys_V2\MainWindow.xaml`, `PowerX.UI\ViewModels\ScriptLibraryViewModel.cs`, `PowerX.UI\ViewModels\ScriptLibraryViewModel.Commands.cs`, `PowerX.Services\Managers\MacroDatabase.cs`

---

## 📋 Comprehensive Audit Checklist

### 1. Dashboard Views & Navigation
- Audit default Dashboard views: **Connections**, **My Macros**, **Text Snippets**, and **Custom Actions**.
- Check all button command bindings: `AddSnippetCommand`, `CreateMacroCommand`, `DeleteMacroCommand`, `SwitchProfileCommand`.
- Audit Gear Menus on saved macro cards: verify **Trigger Mode** (Hotkey vs Schedule vs Event), **App Scope** (Any Window vs Specific Window), and option checkboxes.

### 2. Profile Lifecycle & Cascade Operations
- Audit the **New Profile** popup dialog UI and ViewModel creation logic.
- Audit the Custom Emoji and Icon picker selection flow.
- Audit the **Move Profile** dropdown menu behavior.
- **Critical Database Cascade Verification**: Inspect `MacroDatabase.cs` deletion logic to verify that deleting a profile purges all associated macros, steps, and shortcuts from `macros.db` without leaving orphaned database rows.

### 3. Saved Macros Context Menu & Stats Widget
- Audit all context menu options on saved macro cards: **Open Editor**, **Duplicate**, **Rename**, **Delete**.
- Audit the **Performance Stats** widget calculations (`Time Saved`, `Execution Count`, `Speed Boost`).
- Audit the floating onboarding guide ("Welcome to My Macros").

---

## 📝 Output Requirement
Please write your detailed findings, verified root causes, line numbers, and actionable bug reports directly to:
`c:\Users\Maaz\Documents\New folder\audit_report_1.md`
