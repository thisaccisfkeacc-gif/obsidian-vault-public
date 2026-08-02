# 🔍 Audit Report 1: Dashboard & Profile Lifecycle

**Date:** 2026-07-26  
**Project:** PowerX Keys  
**Scope:** MainWindow.xaml, ScriptLibraryViewModel (.cs, .Commands.cs, .State.cs), MainViewModel.cs, MacroDatabase.cs, ProfileCreationWindow (.xaml, .xaml.cs)

### ✅ Final Verification — All Items Passed
| Item | Status | Note |
|------|--------|------|
| Unicode emoji encoding (ScriptLibraryView.xaml) | ✅ PASS | `⚡`, `📋` render cleanly |
| Drag-to-Profile (`GetTargetProfile` in MainWindow.xaml.cs) | ✅ PASS | Code exists and functional |
| Edit Profile Title (`ProfileCreationWindow.xaml.cs`) | ✅ PASS | `Title` correctly set on edit |

---

## 1. Dashboard Views & Navigation

### Status: ✅ Mostly Solid — 1 Minor Issue

| Check | Verdict | Details |
|-------|---------|---------|
| Dashboard views (Connections, My Macros, Text Snippets, Custom Actions) | ✅ PASS | `CurrentView` ContentControl switches between ScriptLibraryView (profiles/macros), TextSnippets view, etc. |
| `AddSnippetCommand` | ✅ PASS | Wired in MainViewModel, navigates to TextSnippets |
| `CreateMacroCommand` | ✅ PASS | Bound to "Create" button → `NavigateToMacroEditorCommand` |
| `DeleteMacroCommand` | ✅ PASS | Bound in saved macros context menu, proper cascade (DB + hotkeys + in-memory) |
| `SwitchProfileCommand` | ✅ PASS | Left-click on sidebar profile → `ActiveProfile = profileName`, in-place RefreshLibrary or new view |

**🎯 Minor Issue — Sidebar Built-In Profile Icons**  
`MainWindow.xaml` lines 329-338 use `DataTrigger` on hardcoded profile names `"Default"`, `"CustomActions"`, `"MacroBindings"` to set emoji icons (🏠, 🚀, ⚡). These triggers work only for the English display names. If these names are ever localized or changed, the icons silently disappear.

### Saved Macros Context Menu & Gear Menus

**Status: ✅ PASS**

| Menu Item | Command | Bound? |
|-----------|---------|--------|
| Open Editor | `EditMacroCommand` | ✅ (MainWindow.xaml:688) |
| Duplicate | `DuplicateMacroCommand` | ✅ (MainWindow.xaml:693) |
| Rename | `RenameMacroCommand` | ✅ (MainWindow.xaml:698) |
| Delete | `DeleteMacroCommand` | ✅ (MainWindow.xaml:704) |

**DuplicateMacroCommand** (MainViewModel.cs:700-738):
- Copies steps via `s.Clone()` ✅
- Handles naming collisions (`" (Copy)"`, `" (Copy N)"`) ✅
- Uses `Dispatcher.BeginInvoke(Background)` to prevent white flash ✅

**DeleteMacroCommand** (MainViewModel.cs:741-777):
- Safety confirmation dialog when enabled ✅
- Deletes from DB, removes orphaned hotkeys, removes from in-memory Macros collection ✅
- Live-updates ScriptLibraryView to remove ghost cards ✅

---

## 2. Profile Lifecycle & Cascade Operations

### 2a. New Profile Popup (ProfileCreationWindow)

**Status: ✅ PASS — Fully functional**

- **Name input**: TextBox with 30-char limit, duplicate/reserved name validation ✅
- **Icon picker**: 36 preset emojis in ListBox with visual selection ✅
- **Custom emoji**: Hidden RichTextBox + `Win+.` keyboard simulation via `keybd_event` ✅
- **Create button**: Disabled until valid name + prevents duplicate submission ✅
- **Edit mode**: `LoadExisting()` method reuses same window for rename ✅

### 2b. Delete Profile Cascade

**Status: ⚠️ PARTIAL — Orphaned macro risk**

The cascade code in `MainViewModel.cs:891-968` does this:

```
Delete Profile "X":
  1. Remove profile from Profiles collection          ✅
  2. Remove from ConfigManager.Current.Profiles       ✅
  3. Remove from Settings.CustomProfiles              ✅
  4. Find all Hotkeys where AssignedProfile == "X"    ✅
  5. For each matching Hotkey:
     a. Remove from ConfigManager.Current.Hotkeys     ✅
     b. If Category == "Custom Macros" → MacroDatabase.DeleteMacro(macroId) ✅
     c. Remove phantom macro from in-memory Macros    ✅
  6. Save ConfigManager                                ✅
  7. Switch ActiveProfile to "CustomActions"           ✅
```

**🔴 Finding: `AssignedProfile` column MISSING from `Macros` table**

The `Macros` SQLite table schema (created at `MacroDatabase.cs:90-96`) only has:
```
Id TEXT PK, Name TEXT, Icon TEXT
```
+ migrations adding: `IsFavorite`, `PlaybackSpeed`, `MousePhysicsProfile`, `TraceCaptureMode`, `BlockHardwareInput`, `AutoDelayEnabled`, `AutoDelayMs`, `IsHumanized`, `DefaultHumanizationLevel`

**There is NO `AssignedProfile` column on the `Macros` table.** Macros are linked to profiles *only indirectly* through `ConfigManager.Current.Hotkeys[].AssignedProfile` (the hotkey binding).

**Impact:** If a macro exists in the DB but has no corresponding hotkey binding (e.g., created via Macro Editor but never bound to a hotkey), deleting the profile will NOT delete that macro from the database — it becomes an **orphaned row** that's never displayed but still consumes space.

**Severity:** Low (no user-facing crash, just stale data)

### 2c. Rename Profile

**Status: ✅ PASS**

- Updates profile name in Profiles collection and ConfigManager ✅
- Updates all hotkeys' `AssignedProfile` with the new name ✅
- Updates `RunningProfiles` and `ActiveProfile` ✅
- Updates icon dictionary ✅

### 2d. Switch Profile

**Status: ✅ PASS**

```csharp
// MainViewModel.cs:1074-1087
ActiveProfile = profileName;
if (CurrentView is ScriptLibraryViewModel svm)
    svm.RefreshLibrary(profileName);  // In-place refresh (F10 fix)
else
    CurrentView = new ScriptLibraryViewModel(profileName);
```

---

## 3. Performance Stats Widget

**Status: ✅ PASS — Bindings present**

Widget popup at `MainWindow.xaml:797-931` displays:

| Stat | Binding | Data Source |
|------|---------|-------------|
| Time Saved Emoji | `{Binding TimeSavedEmoji}` | MainViewModel |
| Time Saved Shoutout | `{Binding TimeSavedShoutout}` | MainViewModel |
| Macros Fired | `{Binding TotalMacrosExecuted}` | MainViewModel |
| Total Time Saved | `{Binding EstimatedTimeSaved}` | MainViewModel |
| AI Generated | `{Binding TotalMacrosGeneratedByAI}` | MainViewModel |
| Snippets Expanded | `{Binding TotalSnippetsExpanded}` | MainViewModel |

All bindings point to `MainViewModel` properties. Ensure these are populated by `FlushMacroStats()` or similar timer callback.

---

## 4. Floating Onboarding Guide

**Status: ✅ PASS — 3 sections, 3 steps each**

| Section | Trigger Flag | Steps |
|---------|-------------|-------|
| Default (My Macros) | `HasSeenDefaultGuide` | Welcome → Assign hotkey → More control |
| MacroBindings | `HasSeenMacrosGuide` | Welcome to My Macros → Assign hotkey → Press Start |
| Quick Actions | `HasSeenQuickActionsGuide` | Welcome → Add action → Assign hotkey |

Guide steps use `ScriptLibraryViewModel.cs:595-717` with `ShowGuide()`, `UpdateGuideStep()`, `ExecuteNextGuideStep()`, `ExecuteDismissGuide()`. Dismissed guides are persisted via `ConfigManager.Current.Settings.*` flags.

---

## 5. Critical Database Cascade Verification

**MacroDatabase.cs `DeleteMacro()` (line 879-898):**
```csharp
DELETE FROM Macros WHERE Id = $id
```
Relies on SQLite `PRAGMA foreign_keys = ON` + `FOREIGN KEY(MacroId) REFERENCES Macros(Id) ON DELETE CASCADE` on the `MacroSteps` table (line 116). This cascade deletes all `MacroSteps` rows when a `Macros` row is deleted. ✅

**However**: `CaptureLibrary_*` tables have NO foreign key relationship to `Macros`. If a UIElement/Image/Pixel/Window library entry was created from a step that is part of a deleted macro, the library entry **survives**. This is intentional (library entries are shared references).

**`DeleteAllMacros()` (line 900-918):**
```csharp
DELETE FROM Macros  // Cascade deletes all MacroSteps
```
No `AssignedProfile` filter — wipes everything. ✅ for its purpose.

---

## Summary

| Area | Verdict |
|------|---------|
| Dashboard Views & Navigation | ✅ PASS |
| Button Command Bindings | ✅ PASS |
| Gear Menus (Open/Duplicate/Rename/Delete) | ✅ PASS |
| New Profile Dialog | ✅ PASS |
| Emoji/Icon Picker | ✅ PASS |
| Delete Profile Cascade | ⚠️ Minor: orphaned macros without hotkey binding survive |
| Rename Profile | ✅ PASS |
| Switch Profile | ✅ PASS |
| Performance Stats Widget | ✅ PASS (bindings present) |
| Onboarding Guide | ✅ PASS |
| Macro DB Cascade (delete macro → steps) | ✅ PASS |
| Macro DB Cascade (delete profile → macros) | ⚠️ No `AssignedProfile` column on Macros table — indirect linking only |

---

## 4. Additional Deep Re-Scan Findings

### 4a. XAML Structural Issues (ScriptLibraryView.xaml)

| Issue | Lines | Details |
|-------|-------|---------|
| **Dead grid row** | 3-8 | Row 2 (index 2, `Auto`) is declared but never populated — a dead gap in the layout |
| **Redundant resource** | 10, 780 | `BooleanToVisibilityConverter` defined at root Grid.Resources AND nested inside a child Grid.Resources — shadows the root one |
| **Fade overlay broken** | 2044-2046 | Both gradient stops of the fade overlay are the SAME color (`{DynamicResource TokenAppBg}`) — produces flat solid, not a gradient fade. Same at lines 2053-2056 |
| **Trailing whitespace** | 2353-2360 | 7 blank trailing lines at end of file |

### 4b. Character Encoding Corruption in App Switcher Gear Menu

| Line | Expected | Actual |
|------|----------|--------|
| 1751 | `⚡` (lightning bolt) | `âš¡` (garbled) |
| 1770 | `📋` (clipboard) | `ðŸ“‹` (garbled) |

**Root Cause:** The file was saved with wrong encoding for these Unicode characters. The same emojis render correctly elsewhere in the same file (lines 1090, 1109 in the File Launchers gear popup).

### 4c. Popup Stuck-Checked Pattern (All Gear Menus)

| Lines | Issue |
|-------|-------|
| 683+ | Gear menu `Popup` uses `IsOpen="{Binding IsChecked, ElementName=SettingsGearBtn}"` with `StaysOpen="False"`. When the popup closes via outside click, the **ToggleButton remains checked**. Next click toggles it off then on — feels glitchy. |
| 1567 | **Inconsistency:** App Switcher gear popup uses `StaysOpen="True"` while all others use `"False"` |

### 4d. Performance Stats Widget (MainWindow.xaml:795-920)

| Issue | Lines | Details |
|-------|-------|---------|
| **Uses StaticResource** | 800 | `Background="{StaticResource TokenGlassBgBrush}"` — if theme changes at runtime, this won't update. Should be `DynamicResource` |
| **Hardcoded shadow color** | 803 | `DropShadowEffect.Color="#A78BFA"` — not theme-aware |
| **Hardcoded header color** | 808 | `PERFORMANCE STATS` label uses `#5A5C67` |
| **Hardcoded icon colors** | 808+ | Multiple SVG path fills hardcoded: `#FACC15` (yellow), `#38BDF8` (blue), `#D946EF` (magenta), `#A78BFA` (purple) |

### 4e. Onboarding Guide Issues (ScriptLibraryViewModel.cs:595-717)

| Issue | Lines | Details |
|-------|-------|---------|
| **No first-run wizard** | N/A | No standalone WelcomeTour or OnboardingWindow class. Guide is just in-app popups triggered per-profile |
| **Guide dismissed mid-way loses progress** | 700-717 | `ExecuteDismissGuide` sets `HasSeen*` flag, but if user switches views mid-guide (steps 1-3), `IsGuideVisible` resets WITHOUT setting the flag — guide reappears next time |
| **CustomActions guide uses internal name** | 610-613 | Checks `CurrentDisplayProfile == "CustomActions"`, but sidebar displays it as "Quick Actions" — user may not recognize the name |
| **SplashWindow no error handling** | SplashWindow.xaml.cs:12-23 | Just fades out. No error states or timeout — if loading hangs, splash stays forever |

### 4f. Profile Lifecycle Deep Findings

#### Profile Creation Window (ProfileCreationWindow.xaml.cs)

| Issue | Lines | Details |
|-------|-------|---------|
| **Window title not updated in edit mode** | xaml:6, xaml.cs:73 | XAML Title hardcoded to "PowerX Keys - Create Profile". Code-behind changes `TitleBlock.Text` to "Edit Profile" but does NOT update the window `Title` property — taskbar/alt-tab always shows "Create Profile" |
| **Emoji+grid selection conflict** | 221-231, 249-258 | `Submit()` prioritizes `_selectedCustomEmoji` over grid selection. But selecting from grid does NOT clear `_selectedCustomEmoji` — if user typed a custom emoji THEN clicked a grid icon, the old custom emoji wins |
| **Reserved names mismatch** | xaml.cs:146-150 vs MainViewModel.cs:858 | ProfileCreationWindow's `_reservedNames` includes `"My Macros"`, `"Quick Actions"`, `"Text Snippets"` which are display names, not internal profile names. The actual reserved set (`MainViewModel`) is `"Default"`, `"CustomActions"`, `"MacroBindings"`, `"ComingSoon"` |
| **Custom emoji text changed doesn't reset** | 263-287 | `CustomEmojiRtb_TextChanged` sets `IconListBox.SelectedIndex = -1` and overwrites `_selectedCustomEmoji`. But if user clicks the custom emoji box without typing, `TextChanged` won't fire — stale state persists |
| **`Win+.` keybd_event unreliable** | 289-313 | Uses Win32 `keybd_event` to simulate Win+. for OS emoji panel. Fails in RDP, sandboxed environments, or with UIPI blocking |

#### Profile Management in Code

| Issue | Location | Details |
|-------|----------|---------|
| **Drag-to-profile is BROKEN for custom profiles** | MainWindow.xaml.cs:18-23 | `GetTargetProfile` only checks `b.Tag is string` and `b.DataContext is string`. Custom profile borders have `DataContext` bound to `AppProfile` objects and `Tag` to MainViewModel — function always returns `"Default"` for custom profiles |
| **No profile ordering exists** | N/A | No drag-to-reorder, no Move Up/Down, no profile order property. Order determined by `ConfigManager.Current.Settings.CustomProfiles` list order |
| **Move-to-profile only handles CustomMacros** | CustomActionCard.xaml.cs:418-419 | After moving to another profile, code only removes from `svm.CustomMacros`. If action was in `QuickActions`, `FileLaunchers`, etc., it's not removed — appears duplicated |
| **`GetTargetProfile` disallows drops on "Default"** | MainWindow.xaml.cs:30,50,79 | The "Default" profile border explicitly excluded from drag-drop. But "Default" contains built-in cards — why can't user drag a macro there? |

### 4g. MacroDatabase.cs Deep Findings

| Issue | Lines | Details |
|-------|-------|---------|
| **No `Profiles` table** | N/A | Profiles stored in JSON config (`ConfigManager.Profiles`), not SQLite. No DB-level cascade possible |
| **Only one FK exists** | 116 | `FOREIGN KEY(MacroId) REFERENCES Macros(Id) ON DELETE CASCADE` — only on `MacroSteps.MacroId` |
| **No FK on `ParentId`** | 113, 204 | `ParentId` column has no FK constraint — if a parent step is deleted manually, orphaned children can exist in DB |
| **Thread-safety race window** | 881-882 | `ClearCache()` holds `_cacheLock`, then DB op holds `_dbLock`. Between them, another thread could call `LoadAllMacros()`, read old DB state, cache it — after delete completes, cache holds stale data |
| **`PRAGMA foreign_keys` not set on connection init** | 32-49 | `GetConnection()` does NOT set `PRAGMA foreign_keys = ON`. Only `SaveMacro`, `DeleteMacro`, and `DeleteAllMacros` set it. If connection is recreated, FK is OFF until one of those runs |
| **Library cleanup date logic inconsistency** | 1227 vs 1714 | Most library cleanup uses `datetime()` for date arithmetic, but `CaptureLibrary_Windows` uses `julianday()` — functionally same but inconsistent implementation |

### 4h. Text Snippets View Issues (TextSnippetsView.xaml, .xaml.cs, TextSnippetsViewModel.cs)

| Issue | Lines | Details |
|-------|-------|---------|
| **Config save on every property change** | xaml.cs:360 | `SnippetItem_PropertyChanged` calls `ConfigManager.Save()` on EVERY property change — including trivial toggles. Heavy I/O on UI thread |
| **Duplicate trigger disable can recurse** | xaml.cs:289-308 | When duplicate trigger detected, snippet is disabled via `IsEnabled`, which re-enters the PropertyChanged handler. `activeDuplicate` guard prevents infinite loop but pattern is fragile |
| **Space removal unsub/resub pattern** | xaml.cs:278-284 | Temporarily unsubscribes from PropertyChanged to modify trigger, then re-subscribes. If exception occurs in-between, handler lost permanently |
| **No virtualization for snippet list** | xaml:383 | `ItemsControl` loads ALL snippets into visual tree — memory grows linearly with 100+ snippets |
| **Hardcoded hex in dialog builders** | xaml.cs:795,804,813 | `#30D158`, `#0D0E14` colors hardcoded in MultiChoiceBuilderDialog — not theme-aware |
| **Popup close on deactivate loses edits** | xaml.cs:1048-1051 | `Window_Deactivated` closes all popups — unsaved RTB edits in output gear menu are lost on alt-tab |

### 4i. Dead Code & Architectural Issues

| Issue | Location | Details |
|-------|----------|---------|
| **`ToggleScopeCommand` declared but never bound in XAML** | ScriptLibraryViewModel.Commands.cs:280 | Command is initialized but never used in any XAML binding. `CycleScopeModeCommand` is used instead |
| **`CurrentlyBindingAction` is static singleton** | ScriptLibraryViewModel.cs:302-315 | Static property — only one action can be binding at a time across ALL ViewModel instances. If multiple windows exist, they fight |
| **`ExecuteCaptureScreenEventUIElement` is empty stub** | ScriptLibraryViewModel.Commands.cs:1133-1136 | `return;` — not implemented |
| **Code duplication: CaptureApp + CaptureAppByCrosshair** | ScriptLibraryViewModel.Commands.cs:562-618 vs 661-717 | Two methods with near-identical minimize/capture/Toast pattern |
| **Infinite loop risk in ScreenEvent pixel capture** | ScriptLibraryViewModel.Commands.cs:1027-1114 | `while (true)` with no max-iteration guard — if `canceledStudio` stays true, loop never breaks |
| **Guide systems fragmented** | ScriptLibraryViewModel.cs:595 vs TextSnippetsViewModel.cs:198 | Two separate guide implementations with different patterns, no unified first-run tour |
