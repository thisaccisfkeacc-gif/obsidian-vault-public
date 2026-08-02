# Auto-Updater Edge Case Scan

Files reviewed: `AutoUpdateService.cs`, `App.xaml.cs` (launch + `EnsureUpdaterExists`), `PowerX_Updater\Program.cs`, `MainWindow.xaml.cs` (trigger site)

**Flow:** `MainWindow` startup → `CheckForUpdatesAsync()` → GitHub API → download ZIP to `%TEMP%` → next app launch detects ZIP → launches `PowerX_Updater.exe` → waits for exit → replaces files → relaunches.

## Findings

### Medium
1. **Updater never updates itself** – `PowerX_Updater.exe`/`.dll` is skipped during file copy (line 186-187). Old updater stays forever. `EnsureUpdaterExists` only extracts if files are missing entirely. New updater versions from future builds are never deployed.
2. **Partial file list for deletion** – Only 5 files (`PowerX Keys.exe`, `PowerX_Keys_V2.exe`, `PowerX.Core.dll`, `PowerX.UI.dll`, `PowerX.Services.dll`) are explicitly deleted before copy. New/renamed files from the update accumulate as stale cruft. Old files removed in the new version aren't cleaned up.
3. **No user notification** – `UpdateDownloaded` event has zero subscribers. `IsUpdateAvailable` / `IsUpdateDownloaded` are set but never read by any UI code. User has no idea an update is pending and won't know to restart.
4. **Version check only happens at check time, not apply time** – Once the ZIP exists on disk, the next launch applies it unconditionally. If user downgraded or the ZIP is from an older version, it still replaces the app.
5. **`Thread.Sleep(500)` after process exit** – Brittle assumption that 500ms is enough for Windows to release all file locks. On slow/loaded systems, the updater's retry logic (15×300ms) compensates partially.
6. **PID wait timeout only 15s** – If main app takes >15s to shut down (async cleanup, network ops), updater tries to replace locked files. Retry logic (4.5s) may or may not save it.
7. **Hardcoded dev fallback path** – `parentDir\PowerX_Updater\bin\Debug\net10.0\PowerX_Updater.exe` only works for Debug builds. Release builds silently skip this.

### Low
8. **No GitHub API rate-limit handling** – 60 req/hr unauthenticated. Silent fail means update check never retries until restart.
9. **No proxy support** – `new HttpClient()` uses system proxy by default on .NET, but no explicit config or auth.
10. **Fire-and-forget download on app close** – `_ = DownloadUpdateSilentlyAsync(downloadUrl)` – if user closes app mid-download, orphaned `.tmp` file is cleaned on next startup (line 269-273).
11. **ZIP validation is minimal** – Only checks 4-byte PK magic number. Corrupted ZIP passes header check but fails later.
12. **Minimal argument quoting** – `Arguments = $"\"{pendingUpdateZip}\" \"{appDir.TrimEnd('\\')}\""` – quotes around paths but edge cases (trailing slash, Unicode chars in path) could break.
13. **`AutoCheckForUpdates` defaults to `true`** – First-ever launch hits GitHub API. Some users prefer opt-in.
14. **`EnsureUpdaterExists` never overwrites stale updater files** – Once present, updater files are never refreshed from embedded resources.
