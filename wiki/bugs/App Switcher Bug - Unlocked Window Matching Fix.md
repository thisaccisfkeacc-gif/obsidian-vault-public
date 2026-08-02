# 🐛 App Switcher: Unlocked Window Matching Fix

**Date**: 2026-07-28
**Status**: 🟢 Fixed

---

## 🎯 Problem
When **"Lock to This Exact Window"** (`IsStrictInstanceTracking`) is **OFF**, clicking the App Switcher shows:
```
Target app status: NOT found
```

The macro **fails** to activate any open window of that app.

## 🔍 Root Cause
One-character bug in `ScriptCompilerService.cs` (~line 1468).

### The stored Path format:
```
chrome.exe - Google Chrome - My Tab
```

### The broken code:
```csharp
// dashIndex < target.IndexOf(".exe") + 4
// This is WRONG — dash always comes AFTER .exe, not before!
```

### What happens:
1. `dashIndex < 10` evaluates to **false**
2. The `if` block is **skipped entirely**
3. Falls through to: `target = "ahk_exe " + target`
4. Produces: `ahk_exe chrome.exe - Google Chrome - My Tab` ❌
5. AHK's `WinExist()` **cannot parse** this malformed string

### The fix:
Changed `<` to `>`
```csharp
// BEFORE (broken)
dashIndex < target.IndexOf(".exe") + 4

// AFTER (fixed)
dashIndex > target.IndexOf(".exe") + 4
```

### Result:
Now correctly generates:
```
Google Chrome - My Tab ahk_exe chrome.exe ✅
```
AHK can match this properly!

---

## 📄 File Changed
- **File**: `PowerX.Services/Services/ScriptCompilerService.cs`
- **Location**: The `AppBound` case in `case "AppBound":` block
- **Change**: Single character — `<` → `>`

## ✅ Build Result
- `dotnet build`: **0 errors**, only warnings (pre-existing)
