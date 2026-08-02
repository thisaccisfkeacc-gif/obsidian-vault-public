# PowerX Keys — Modularization Scan Report (Full)

**Scan Date:** July 24, 2026
**Scanner:** opencode (mimo-v2-pro) — 4 parallel agents
**Scope:** PowerX.Core, PowerX.Services, PowerX.UI, Original Project

---

## Overall Status

| Project | Build | Errors | Warnings | Verdict |
|---------|-------|--------|----------|---------|
| PowerX.Core | ✅ 0 errors | 0 | 8 | ⚠️ Needs namespace cleanup |
| PowerX.Services | ✅ 0 errors | 0 | 8 | ⚠️ Needs namespace cleanup |
| PowerX.UI | ❌ 44 errors | 44 | 26 | ❌ Broken — needs fixes |
| Original Project | ✅ Clean | 0 | 2 | ✅ Done |

---

## PowerX.Core (12 files)

### ✅ Good
- Build succeeds (0 errors, 0 warnings)
- No circular dependencies
- No dead/unused files
- All mutable properties call OnPropertyChanged
- All static delegates are null-guarded
- No IDisposable issues
- No async issues

### ❌ Errors

**1. Split namespaces (6 old, 6 new)**
| Namespace | Files |
|-----------|-------|
| `PowerX.Core.Models` | AppConstants, VersionInfo, ObservableRangeCollection, AIChatMessage, ActionModels |
| `PowerX_Keys_V2.Models` | ViewModelBase, UpdateInfo, MacroItem, CaptureLibraryEntry, AppEnums, AppConfig |
| `PowerX_Keys_V2.Interfaces` | IMacroDatabase |

**2. WPF types in Core**
- `MacroItem.cs`: `SolidColorBrush`, `Color`, `ColorConverter` (lines 1895-1944)
- `MacroItem.cs`: `Application.Current?.Dispatcher?.BeginInvoke` (lines 1370, 1385)
- `CaptureLibraryEntry.cs`: `ImageSource` (line 391)
- `AppConfig.cs`: `Application.Current?.MainWindow` (line 1083)

**3. Duplicate classes**
- `ViewModelBase` — exists in Core AND UI (identical)
- `UpdateInfo` — exists in Core AND Services (identical)

### ⚠️ Warnings
- God files: AppConfig (1701 lines), MacroItem (2550 lines)
- Two separate `GlobalStepChanged` static events (MacroStep + MacroItem)
- Static events can cause memory leaks if not unsubscribed
- `MacroStep` reimplements INPC instead of inheriting ViewModelBase (intentional — has filtering logic)

---

## PowerX.Services (29 files)

### ✅ Good
- Build succeeds (0 errors)
- No ViewModel references (only in comments)
- No circular dependencies
- No duplicate files with original project
- ServicesUIHooks.cs properly decouples UI via static delegates

### ❌ Errors

**1. All 29 files use old namespaces**
- `namespace PowerX_Keys_V2.Services` — 25 files
- `namespace PowerX_Keys_V2.Managers` — 4 files
- Zero files use `namespace PowerX.Services`

**2. IMacroDatabase interface is orphaned**
- `MacroDatabase` is a `static class` — cannot implement interface
- Method `GetMacroById(string)` doesn't exist in implementation

### ⚠️ Warnings
- NuGet security vulnerabilities:
  - `Microsoft.IdentityModel.JsonWebTokens` 7.0.3 — moderate
  - `SQLitePCLRaw.lib.e_sqlite3` 2.1.11 — high
  - `System.IdentityModel.Tokens.Jwt` 7.0.3 — moderate
- ServicesUIHooks.cs uses WPF types in delegate signatures
- `SupabaseAuthService` hardcodes `"PowerXKeys"` instead of `AppConstants.AppNameNoSpaces`

---

## PowerX.UI (148 files)

### ✅ Good
- All 31 converters properly implement IValueConverter/IMultiValueConverter
- All 5 template categories present
- All ViewModels use INotifyPropertyChanged properly
- No circular project references
- .csproj references both PowerX.Core and PowerX.Services
- No duplicate class definitions within UI

### ❌ Errors (44 build errors)

**Missing types (root cause: namespace migration incomplete):**
| Missing Type | Occurrences |
|-------------|-------------|
| `MainWindow` | 8 |
| `App` | 8 |
| `AppConstants` | 6 |
| `EasterEggService` | 5 |
| `UIElementCaptureService` | 4 |
| `TrayIconManager` | 3 |
| `AppTheme` / `ThemeService` | 2 |
| `AIChatMessage` | 1 |

**100% of files use old `PowerX_Keys_V2.*` namespace. Zero files use `PowerX.UI.*`.**

### ⚠️ Warnings (26 warnings)

**Dead files (5):**
- `MacroEditorViewModel.Commands.cs.bak`
- `SearchTemplates.xaml.temp`
- `MacroEditorView.xaml.test`
- `SettingsDashboardView.xaml.cs.clean`
- `SettingsDashboardView.xaml.cs.txt`

**WPF anti-patterns (SEVERE):**
- 90+ direct `new Window()` calls from ViewModels
- `DarkMessageBoxWindow.Show()` called from 7+ ViewModels
- No dependency injection — all services via static methods
- `MainViewModel.Instance` static singleton creates hidden coupling
- God-class ViewModels: MacroEditorViewModel (7 partial files, 1400+ lines), SettingsDashboardViewModel (2117 lines)
- Fire-and-forget `async void` methods
- ViewModel directly writes to Windows Registry
- ViewModel directly manages Process restart

---

## Original PowerX_Keys_V2 (55 files)

### ✅ Good
- Views folder: Deleted
- ViewModels folder: Deleted
- .csproj references all3 projects
- App.xaml and MainWindow.xaml properly reference new projects
- Themes folder: Pure token definitions, no namespace issues
- Resources folder: All properly referenced

### ⚠️ Warnings
- `Services/FileAssociationService.cs` still present (intentional — startup service)
- `build_errors.txt` — orphaned debug artifact
- `PowerX_Updater.pdb` — orphaned PDB file

---

## Priority Fix List

| # | Fix | Project | Risk | Effort |
|---|-----|---------|------|--------|
| 1 | Fix 44 build errors in PowerX.UI | UI | 🔴 High | 🔴 High |
| 2 | Migrate all namespaces to `PowerX.Core.*` / `PowerX.Services.*` / `PowerX.UI.*` | All | 🔴 High | 🔴 High |
| 3 | Remove duplicate ViewModelBase from UI | UI | 🟢 Low | 🟢 Low |
| 4 | Remove duplicate UpdateInfo from Services | Services | 🟢 Low | 🟢 Low |
| 5 | Move WPF types out of Core | Core | 🔴 High | 🔴 High |
| 6 | Fix IMacroDatabase orphaned interface | Services | 🟡 Medium | 🟡 Medium |
| 7 | Update NuGet packages for security | Services | 🟢 Low | 🟢 Low |
| 8 | Delete 5 dead files from UI | UI | 🟢 Low | 🟢 Low |
| 9 | Split god files (AppConfig, MacroItem) | Core | 🔴 High | 🔴 High |
| 10 | Move FileAssociationService to Services | Original | 🟢 Low | 🟢 Low |

---

## Labels Key

- ✅ **Good** — No issue
- ⚠️ **Warning** — Potential issue, should fix
- ❌ **Error** — Definite issue, must fix
- 📌 **Intentional** — Looks odd but is by design

---

**Last Updated:** July 24, 2026
