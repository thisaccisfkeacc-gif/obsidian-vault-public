---
tags: [guide, packaging, obfuscation, deployment]
date: 2026-08-01
status: active
---

# Packaging & Obfuscation Guide 📦

**Summary:** Overview of the production build, code obfuscation (using the Obfuscator/Obsequator), and final packaging process for PowerX Keys before release.

## Production Build & Obfuscation Workflow

When the application is ready for release, follow this sequence to compile, scramble, and package the code:

### 1. Release Compilation
Always build the application in **Release** mode to optimize performance, remove debug symbols, and strip developer logs:
```bash
dotnet publish -c Release -r win-x64 --self-contained true
```

### 2. Code Obfuscation (The Obfuscator / Obsequator)
To prevent reverse-engineering and protect source code integrity:
* Run the code obfuscator/obsequator utility against the output DLLs (specifically `PowerX Keys.dll` and `PowerX_Updater.dll`).
* **Scrambling Rules:**
  * Scramble class and method names.
  * Encrypt/obfuscate sensitive string constants (like the Supabase Edge Function API URLs or local encryption keys).
  * Control-flow obfuscation to mangle decompiled logical trees.
* **Exclusions (Crucial):**
  * Ensure that JSON model classes (e.g., `MacroStep`, `UpdateInfo`, `SettingsModel`) are **excluded** from property name scrambling. Scrambling their names will break JSON serialization/deserialization when loading macros or settings!
  * Exclude WPF code-behind classes and XAML-bound properties (e.g., `DisplayMacroSteps`, `CurrentMacro`) to prevent data-binding failures.

### 3. Release Formats & Packaging
We will compile and distribute the application in two target formats:
1. **Executable Installer (Setup `.exe`):**  
   * **Behavior:** A standard setup wizard installer (built using Inno Setup or Advanced Installer).
   * **User Experience:** One-click and boom! It automatically extracts files to `LocalAppData` or `Program Files`, registers global shortcut paths, configures the updater executable, creates Desktop/Start Menu shortcuts, and launches the app.
2. **Portable Version (Standalone `.zip`):**  
   * **Behavior:** A raw zipped directory of the compiled/obfuscated bin folder.
   * **User Experience:** Completely portable. Users just unzip it into any folder and run `PowerX Keys.exe` directly. No system installation, registry changes, or admin permissions required.

---

## MANDATORY: Dynamic Scan Before Cleanup

**DO NOT blindly follow the static checklist below.** The checklist may be outdated.

Before clearing any data or files, the agent MUST:

1. **Scan the current codebase** for any new data stores, caches, auth mechanisms, config files, or local state that have been added since the checklist was last updated.
   - Check for new `.db` files, `.json` configs, credential stores, token caches, session files, local storage folders, etc.
   - Look at recently added services (auth, licensing, analytics) to identify any new local artifacts they create.
2. **Compare findings against the checklist below.** If anything new is found that should be cleared before packaging, **add it to the checklist** in this file.
3. **Present the full updated cleanup list to the user** — show what's already listed AND what was newly discovered.
4. **Wait for user approval** before proceeding with any deletion or clearing.
5. Only after approval: execute the cleanup, then continue to packaging.

---

## 🧹 Crucial Production Clean-Up Checklist

Before publishing either the Setup or ZIP bundle, **ALWAYS remove the following development files and directories** from the build directory to keep the release clean and private:

| Target File / Folder | Action | Why it must be removed |
|---|---|---|
| `/TempImages/` | **DELETE** | Contains local screenshots and temporary captured element graphics from dev testing. |
| `%LOCALAPPDATA%/PowerXKeys/debug_log.txt` | **DELETE** | Deletes AutoHotkey runtime debug output logs and developer tracing prints. |
| `%LOCALAPPDATA%/PowerXKeys/Configs/macros.db` (and JSON backups) | **DELETE / REPLACE** | Remove your local database file containing personal test macros and execution history. Let the app auto-generate a fresh, empty SQLite DB file on first user startup. |
| `%LOCALAPPDATA%/PowerXKeys/config.json` (Local settings) | **DELETE** | Deletes local configuration states, dev testing coordinates, and custom hotkey bindings. The app will auto-generate factory settings on first run. |
| `.pdb` files | **REMOVE** | Program Debug Database files contain original code line mappings and directory structures, which makes reverse-engineering easy. |
| `.env` file | **DELETE / EXCLUDE** | Contains sensitive developer credentials, Supabase service roles, and private API keys. It must **never** be packaged or shipped to users. |
| Supabase auth tokens / session data | **CLEAR** | Any cached login sessions, refresh tokens, or user profile data from dev testing. App should start fresh with no logged-in user. |
| Windows Credential Manager entries | **CLEAR** | If the app stores any credentials via Windows Credential Manager, remove test entries so end users start clean. |
| First-run / onboarding flags | **RESET** | Ensure `IsFirstRun`, onboarding completion flags, and trial counters are not baked into shipped settings. App must trigger first-run experience for new users. |
| License / activation cache | **CLEAR** | Any locally cached license keys, activation timestamps, or plan tier data from dev testing must not ship. |

## Related Pages
- [[overview]]
- [[auto-update]]
- [[settings-dashboard]]
