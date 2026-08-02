# 🔍 Deep Audit Prompt 2: Settings Panel & Light Mode Color Theme Audit

You are tasked with executing a deep structural and visual audit of the **PowerX Keys** Settings Dashboard and Light Mode theme system.

---

## 💡 Research Strategy
- **Search Obsidian Vault First**: Search `c:\Users\Maaz\Documents\New folder\Obsidian Vault\` for existing documentation, GOTCHAS, and schemas before doing full codebase searches.
- **Use Graphify Knowledge Graph**: Check `graphify-out/` or use `graphify query` for fast node/relationship lookup across components.

---

## 🎯 Target Scope
- **Root Directory**: `c:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\`
- **Core Files**: `PowerX.UI\Views\SettingsDashboardView.xaml`, `PowerX.UI\ViewModels\SettingsDashboardViewModel.cs`, `PowerX.UI\ViewModels\SettingsDashboardViewModel.Commands.cs`, `PowerX.Services\Services\ConfigManager.cs`

---

## 📋 Comprehensive Audit Checklist

### 1. Settings Dashboard Deep Scan
- Audit all settings categories: **General**, **Automation & Engine**, **Hotkeys**, **Account & Licensing**, **Advanced Settings**.
- Inspect all toggles, sliders, textboxes, and dropdowns for unhandled exceptions, invalid default fallbacks, or missing tooltips.
- Inspect Account section: trial progress bar binding (`TrialDaysRemaining`), cached subscription status sync, and Upgrade button routing.

### 2. Light Mode Theme Audit
- Deep audit Light Mode color theme consistency across `SettingsDashboardView.xaml`, `MacroEditorView.xaml`, `MainWindow.xaml`, and `MacroStepCard.xaml`.
- Search for hardcoded hex colors (e.g., `#1E1E1E`, `#121212`), un-themed text labels, broken contrast ratios, or missing `DynamicResource` token bindings.
- Provide a clear recommendation: Provide full color token fixes OR recommend cleanly hiding/disabling Light Mode in favor of the default sleek dark theme.

---

## 📝 Output Requirement
Please write your detailed findings, verified root causes, line numbers, and actionable bug reports directly to:
`c:\Users\Maaz\Documents\New folder\audit_report_2.md`
