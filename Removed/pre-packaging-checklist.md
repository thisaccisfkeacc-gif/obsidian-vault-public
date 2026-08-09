# 📦 Pre-Packaging Checklist — PowerX Keys

> Everything that MUST be done before building the installer and going live.
> An agent can work through this list when Maaz is ready to ship.

---

## 🔗 1. Update the Supabase Download Link

**File:** Supabase Edge Function `check-updates` (project: `sgqyjylychviegbyygps`)

Right now the download link is a placeholder:
```json
"downloadLink": "https://github.com/maazvfx/PowerX-Keys/releases/latest"
```

**What to do:**
- Build the release installer (`.exe` or `.zip`)
- Upload it to GitHub Releases OR a personal website
- Update `downloadLink` in the Supabase function to the real direct download URL
- Update `latestVersion` to the correct version number (e.g. `"5.3.0"`)

---

## 🔢 2. Confirm the Version Number

**File:** `Models/VersionInfo.cs` (or wherever `VersionInfo.FullVersion` is set)

Make sure the version string in the app matches what's in Supabase exactly.
If they don't match, the update check will always show "update available" on first launch.

---

## 🐛 3. Fix Known Bugs (Desktop Bug Report)

**File:** `C:\Users\Maaz\Desktop\PowerX_Keys_Bugs.md`

There are 7 open bugs documented here from the Settings Dashboard audit.
These should be fixed before publishing so users don't encounter them.

---

## 🔴 4. Monetization Lever — Keep OFF

**File:** Supabase `check-updates` function

Make sure `forceUpdate` stays `false` for the initial free release:
```json
"forceUpdate": false
```
Only flip this to `true` when switching to the paid model in the future.

---

## 🧹 5. Remove Debug/Dev Artifacts

- Remove any hardcoded test keys, test URLs, or debug-only `Debug.WriteLine` calls
- Confirm `debug_log.txt` is NOT bundled in installer (it is created at runtime at `%LOCALAPPDATA%\PowerXKeys\debug_log.txt`)
- Keep local debug logging enabled by default (auto-trimmed at 1 MB / 500 KB on launch)
- Crash reporting automatically attaches last 50 lines of debug log for full diagnostic context
- Confirm the AI API key is loaded from config, not hardcoded

---

## 🏪 6. App Store / Distribution Setup

- Create a proper GitHub Release with release notes
- Upload the installer binary
- Test the installer on a clean Windows machine (no dev tools installed)
- Confirm the app runs without Visual Studio / .NET SDK installed (self-contained publish)

---

## ✅ Done When Ready to Ship

Once all the above are ticked off, the app is ready for public distribution.
The update system, force-update lever, and all core features are fully wired up and waiting.
