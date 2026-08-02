---
name: release-packaging
description: Build, obfuscate, and package PowerX Keys for release. Use when preparing a new version for distribution — from Release compilation to obfuscation, clean-up checklist, and GitHub deployment for auto-update.
tags: [skill, release, packaging, deployment, obfuscation]
date: 2026-06-08
status: active
sources:
  - wiki/guides/packaging-and-obfuscation.md
  - wiki/guides/github-setup.md
  - wiki/services/auto-update.md
---

# 📦 Skill: Release & Packaging

This skill covers the full release pipeline for PowerX Keys — from version bump to final GitHub deployment.

## Pre-Release Checklist

Before starting a release build, confirm:
- [ ] All 🔴 CRITICAL bugs in `wiki/status/pre-ship-critical-bugs.md` are fixed
- [ ] QA checklist (70 tests) has no new ❌ Fail results
- [ ] `wiki/status/current-version.md` is up to date
- [ ] Version numbers are consistent (see Version Number Rule below)

## Step 1: Version Number Update

**Single source of truth — only ONE place to update:**
- `VersionInfo.cs` — the display version AND the runtime comparison version

`AutoUpdateService.cs` reads directly from `Models.VersionInfo.FullVersion` at runtime — no separate constant to maintain.

```csharp
// VersionInfo.cs — THE ONLY FILE TO EDIT
public const string Major = "5";
public const string Minor = "4";
public const string Patch = "0";
```

**Also update `.csproj` metadata** (for Windows File Properties → Details tab):
```xml
<Version>5.4.0</Version>
<AssemblyVersion>5.4.0.0</AssemblyVersion>
<FileVersion>5.4.0.0</FileVersion>
```

**Version scheme:** Major.Minor.Patch (e.g., 5.4.0)

## Step 2: Release Build

```powershell
# Self-contained Windows x64 release build
dotnet publish -c Release -r win-x64 --self-contained true `
  -p:PublishSingleFile=false `
  "c:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\PowerX_Keys_V2\PowerX_Keys_V2.csproj"
```

Output will be in: `PowerX_Keys_V2\bin\Release\net10.0-windows\win-x64\publish\`

## Step 3: Code Obfuscation

Run the Obfuscator/Obsequator against the published output.

**Scramble these:**
- Class and method names (non-public)
- Sensitive string constants (Supabase URLs, encryption keys)
- Control flow (mangle decompiled logic)

**EXCLUDE from scrambling (critical):**
- `MacroStep` — JSON serialization breaks if scrambled
- `UpdateInfo` — JSON deserialization breaks
- `SettingsModel` — JSON config breaks
- WPF XAML-bound properties (data binding breaks)
- Any class with `[JsonProperty]` attributes

## Step 4: Pre-Package Clean-Up

### MANDATORY: Dynamic Scan Before Cleanup

**DO NOT blindly follow the static checklist below.** The checklist may be outdated.

Before clearing any data or files, the agent MUST:

1. **Scan the current codebase** for any new data stores, caches, auth mechanisms, config files, or local state that have been added since the checklist was last updated.
   - Check for new `.db` files, `.json` configs, credential stores, token caches, session files, local storage folders, etc.
   - Look at recently added services (auth, licensing, analytics) to identify any new local artifacts they create.
2. **Compare findings against the checklist below.** If anything new is found that should be cleared before packaging, **add it to the checklist** in both this file AND `wiki/guides/packaging-and-obfuscation.md`.
3. **Present the full updated cleanup list to the user** — show what's already listed AND what was newly discovered.
4. **Wait for user approval** before proceeding with any deletion or clearing.
5. Only after approval: execute the cleanup, then continue to packaging.

---

**Static checklist (kept updated, but always verify with scan above):**

| File/Folder | Why Remove |
|------------|-----------|
| `/TempImages/` | Dev screenshots — private |
| `debug_log.txt` | Dev AHK debug output |
| `powerx_keys.db` | Your personal test macros |
| `settings.json` | Your dev settings/hotkeys |
| `*.pdb` | Debug symbols — reverse-engineering risk |
| `.env` | ⚠️ Credentials — MUST NEVER ship |
| Supabase auth tokens / session data | Cached login sessions and refresh tokens from dev testing |
| Windows Credential Manager entries | Any stored credentials for the app from testing |
| First-run / onboarding flags | Must not be pre-set — new users need the first-run experience |
| License / activation cache | Dev license data must not ship |

## Step 5: Build Two Packages

### Installer (.exe)
- Use Inno Setup or Advanced Installer
- Auto-extracts to `%LOCALAPPDATA%`
- Creates Desktop/Start Menu shortcuts
- Registers updater executable
- Launches app after install

### Portable (.zip)
- Raw zipped bin folder (no installation)
- User unzips and runs `PowerX Keys.exe` directly
- No registry changes, no admin needed

## Step 6: GitHub Release

See `wiki/guides/github-setup.md` for exact CLI commands.

1. Commit and tag: `git tag v5.3.1`
2. Push the tag: `git push origin v5.3.1`
3. Create GitHub Release with tag
4. Upload both packages as release assets
5. Auto-update system reads the release from GitHub Pages

## Step 7: Update the What's New Page

See `wiki/guides/whatsnew-update-guide.md` for the step-by-step.

## Step 8: Post-Release Wiki Updates

After release:
- Update `wiki/status/current-version.md` with new version + release notes
- Append to `wiki/log.md`
- Clear any stale entries in `wiki/status/pre-ship-critical-bugs.md`

## Rollback Plan

If a critical bug is found post-release:
1. Unpublish/hide the release on GitHub
2. Fix the bug → increment patch version (e.g., 5.3.2)
3. Re-run this full workflow
4. Never overwrite an existing release — always new version

## Related Pages

- [[packaging-and-obfuscation]] — Full packaging guide
- [[auto-update]] — How the auto-update mechanism works
- [[github-setup]] — GitHub CLI and repo security
- [[whatsnew-update-guide]] — Website update instructions
- [[current-version]] — Version history
