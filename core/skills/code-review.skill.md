---
name: code-review
description: Review C#, WPF, and AutoHotkey code in PowerX Keys for correctness, safety, thread safety, MVVM violations, and project conventions. Use before finalizing any code change, or when asked to review a pull request or diff.
tags: [skill, code-review, csharp, wpf, ahk, quality]
date: 2026-06-08
status: active
---

# 🔍 Skill: Code Review for PowerX Keys

This skill provides a structured review checklist tuned to the PowerX Keys tech stack (C# .NET 10, WPF, WinForms, AHK v2, SQLite).

## Review Checklist

### ✅ 1. Thread Safety
- [ ] Any UI property mutation from non-UI thread? → Must use `Dispatcher.Invoke`
- [ ] Any `async void` methods that modify shared state? → Consider wrapping in try-catch
- [ ] Any access to `ObservableCollection` from a background thread? → Must Dispatcher-dispatch
- [ ] Any `static` mutable fields shared across threads? → Check for missing locks
- [ ] `ScriptManager` process operations called from UI thread? → Should be async

### ✅ 2. MVVM Architecture
- [ ] Logic in code-behind (`.xaml.cs`)? → Should be in ViewModel
- [ ] Direct UI manipulation from ViewModel (e.g., `MessageBox.Show`)? → Use a dialog service
- [ ] Missing `OnPropertyChanged()` in property setters? → UI won't update
- [ ] Commands wired correctly (`CanExecute` refreshes when state changes)?

### ✅ 3. Null Safety
- [ ] Any `?.` or null check missing before using a potentially null object?
- [ ] Any indexer access (`list[0]`) without bounds check?
- [ ] `ConfigManager.Current` accessed before initialization?
- [ ] `MacroDatabase` called before the database file exists?

### ✅ 4. AHK / ScriptCompiler
- [ ] Any changes that affect generated AHK syntax? → AHK v2, not v1 syntax
- [ ] String interpolation correct? (C# `$""` on C# side, AHK `.` concat on AHK side)
- [ ] New step type added but not handled in `CompileMasterScript()`? → Will silently skip
- [ ] Engine reload triggered after changes? → `ScriptManager.Stop()` then `Start()`

### ✅ 5. Data Persistence
- [ ] New setting added to `SettingsModel` but not initialized with a default?
- [ ] SQLite operation without error handling? → Can crash on locked DB
- [ ] Config saved immediately rather than using the 500ms debounce? → Can cause save storms
- [ ] New SQLite column added without migration? → Will crash for existing users

### ✅ 6. PowerX Keys Conventions
- [ ] Using `List<T>` for UI-bound collection? → Should be `ObservableCollection<T>`
- [ ] Editing a generated `.ahk` script directly? → ❌ Never — edit `ScriptCompilerService.cs`
- [ ] Force-kill of any process added? → ❌ Removed for compliance (AgDR-003)
- [ ] Version number inconsistency? → Check both `VersionInfo.cs` and `AutoUpdateService.cs`
- [ ] Wild-card `catch (Exception)` swallowing errors silently? → Log or rethrow

### ✅ 7. Security
- [ ] User input used directly in AHK script generation without sanitization? → Script injection risk
- [ ] File path from user input without validation? → Path traversal risk
- [ ] Credentials or API keys hardcoded? → Use config/environment
- [ ] Remote server endpoint exposed without PIN auth? → Check `RemoteServerService`

### ✅ 8. Performance
- [ ] Heavy operation (file I/O, DB query) on UI thread? → Use `Task.Run`
- [ ] SQLite query in a loop? → Batch it
- [ ] Observable collection updated in a loop? → Batch with `AddRange` pattern or suspend notifications
- [ ] Image loaded in memory that could be streamed?

## Severity Labels

| Label | Meaning |
|-------|---------|
| 🔴 CRITICAL | Will crash, data loss, security issue — must fix before ship |
| 🟠 IMPORTANT | Likely bug, performance issue, MVVM violation — fix this release |
| 🟡 MINOR | Code smell, style issue, future risk — log and move on |
| 💡 SUGGESTION | Optional improvement — non-blocking |

## Output Format (When Writing a Review)

```markdown
### [🔴/🟠/🟡/💡] Issue Title
- **File:** filename.cs — Line NNN (or "XAML: ViewName.xaml")
- **Type:** thread-safety | null | mvvm | ahk | persistence | security | perf | convention
- **What:** One sentence — what's wrong
- **Why:** One sentence — why it's a problem
- **Fix:** What should be done instead
```

## What NOT to Nitpick

- Variable naming (unless extremely misleading)
- Minor style differences from your personal preference
- Things that work fine but "could be improved" — scope is correctness, not perfection
- Pre-existing issues you weren't asked to fix → log them separately

## Related Pages

- [[win32-keyboard-gotchas]] — Keyboard-specific pitfalls
- [[execution-pipeline]] — AHK flow to review against
- [[dual-execution-model]] — When AHK vs C# P/Invoke is used
