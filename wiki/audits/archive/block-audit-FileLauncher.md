# Block Audit: FileLauncher (Deep — Cross-Path)

**Date:** 2026-07-24
**Scope:** C# in-process, AHK compiled, AHK preview (SingleStep)

---

## BUGS

### 1. AHK main compile path lacks URL detection
- **Severity:** Medium
- **Location:** ScriptCompilerService.cs:3111-3127
- **Problem:** Sandbox path explicitly checks for `http://`, `https://`, `ftp://` prefixes. Main compile path does not — URLs passed through `DirExist()` (returns false) then `Run()` which may or may not work.

### 2. C# preview silently swallows launch errors
- **Severity:** Low
- **Location:** MacroExecutionService.cs:1607-1619
- **Problem:** `catch { }` discards all exceptions including `Win32Exception` with useful error codes. No user feedback.

### 3. Percent signs in paths not escaped
- **Severity:** Low
- **Location:** ScriptCompilerService.cs:3113
- **Problem:** Only `"` → `""` escaping applied. Percent signs (`%`) in paths cause AHK variable expansion errors.

---

## DEAD CODE

1. Sandbox CompileSandboxStep has its own FileLauncher Run() — 3rd implementation path. Lacks directory detection and URL support.

---

## INCONSISTENCIES

1. C# preview uses `Process.Start` with `UseShellExecute=true`. AHK does `DirExist` + `explorer.exe` for dirs.
2. Post-launch delay: C# has 800ms. AHK has none explicitly.
3. TestFileLauncher (ViewModel) shows error dialog. ExecuteStepAsync silently catches.

---

## VERIFIED OK

- Directory detection via `DirExist()` — correct
- `UseShellExecute = true` in C# — correct
- `LastActionSucceeded` tracking in AHK — correct
