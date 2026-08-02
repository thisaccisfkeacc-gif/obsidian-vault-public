---
tags: [wiki, log]
date: 2026-05-23
status: active
---

# Wiki Change Log

> Append-only log of all wiki changes. Most recent entries at the bottom.
> 
> **Trim Rule:** Every 2 weeks, archive entries older than 14 days into a single summary block at the top. Keep only the last 2 weeks of detailed entries. This prevents the log from growing past ~100 lines.

---

## [ARCHIVED] Summary through 2026-07-09

- **Jul 5–6**: WIN_SMART scope merged, gray-out UI elements, Human Flow rebrand + search scope labels
- **Jul 7**: Removed hidden layout/smart mode features (-1201 lines), version badge fix
- **Jul 8**: Fixed 10+ issues — popup mode default, dynamic step names, ComboBox crash, dynamic dropdown sync, hardcoded block names, monochrome emojis, UIElement preview visibility, duplicate name warning, move submenu repopulation. Configured HANDOFF/AGENTS session startup.
- **Jul 9**: Core agent files cleanup (DECISIONS, GOTCHAS, SOUL, AGENTS, SCHEMA trimmed). Future plans cleanup. Wiki index major overhaul (removed 30+ dead links, synced skills library).

---

## 2026-07-10 - Wiki Guides Directory Cleanup

- Fixed broken links in adding-a-feature.md, agent-onboarding.md, debug-log-strategy.md, win32-keyboard-gotchas.md

## 2026-07-10 - Fixed Macro Editor Preview Button

- **MacroEditorViewModel.Commands.cs** - UnifiedPreviewCommand now plays entire macro when parameter is null

## 2026-07-10 - Added Obsidian-First Search Rule + Expanded AGENTS.md

- Added "Search Obsidian First", "Check GOTCHAS First", "Targeted Scope & Bug Logging" rules

## 2026-07-10 - Optimized MSBuild Build Speed

- **PowerX_Keys_V2.csproj** - Added Inputs/Outputs to CopyUpdaterBinaries target for dependency tracking

## 2026-07-10 - Multiple UI Fixes

- Hidden Preview Step for empty Popup/Notification blocks
- Fixed warning dot on multiple unnamed blocks
- Fixed Group/Repeat naming and Move Menu loop icon
- Fixed Type Text block card data binding (TwoWay mode + emoji TextBlock)
- Added Edit Coordinates to context menu
- Added master switch for AI Assistant
- Fixed context menu Edit Coordinates crash (RoutedCommands)
- Registered token-optimization and context-compression workspace skills
- Hidden settings gears & preview options for empty blocks
- Fixed context menu block numberings
- Fixed File Launcher properties panel visibility

## 2026-07-11 - Unified App Version Source

- Removed hardcoded version, reads from assembly now

## 2026-07-12 - Multiple Fixes

- Fixed Window Pinpoint left-click capture
- Fixed Edit Coordinates input handling (validation, clamping)
- Removed redundant Pinpoint pill from Pixel block
- Set Pinpoint search scope for Pixel Studio capture
- Fixed Play Sound block skipping bug (added AHK wait parameter)
- Always On Top flicker and border softening

## 2026-07-13 - Account Switcher IDE Checks

- Added isIDERunning check to switch/clear/save flows, loading overlay, graceful shutdown

## 2026-07-14 - Major UI Overhaul Day

- Fixed spacing in Window block inline template
- Filter hidden virtual desktop windows from hover highlighting
- Split combined Window block buttons
- Fixed App Switcher capture/list button bindings
- Always on Top flicker fix (atomic Win32 SetWindowPos + caching)
- Always on Top DPI highlight scaling fix
- Fixed UI Element capture highlight stuck issue
- Simplified App Switcher card UI
- Hid App Switcher gear settings
- Fixed Window capture highlight stuck bug
- Matched App Switcher highlight with Window block (purple)
- Restored Size setting & updated color
- Merged File Launcher gear popups
- File Launcher URL validation
- App Switcher Restore shows related settings
- Clarified App Switcher strict matching
- File Launcher menu order + Browse label
- Fixed App Switcher Restore checkmark & removed fresh sizing prompt
- URL-Only Launcher + App Switcher window title display
- Dynamic App Switcher strict window display, title trimming & UI cleanup

## 2026-07-15 - Settings Dashboard & Editor Cleanup

- Fixed settings dashboard crash (invalid style resource)
- Removed redundant auto-bind image target checkbox from editor gear

## 2026-07-15 - Undo/Redo Edge Case Audit

- Added wiki/features/undo-redo-audit.md — 2 critical, 4 medium, 4 low findings

## 2026-07-15 - Input Safety Audit

- Added wiki/audits/input-safety-audit.md — fixed 5 TextBoxes across 4 files

## 2026-07-15 — Index & Schema Cleanup + Kiro Integration

- index.md: Removed 8 dead links, added 10 missing pages, added Audits section
- SCHEMA.md: Added audits/ folder
- SOUL.md: Trimmed to core values + decision framework only
- AGENTS.md: Removed duplicated rules, kept project context only
- Kiro steering: Set up 4 auto-loading steering files connected to Obsidian

## 2026-07-16 — Removed Search Cascade UI & Fixed Double-Click Error

- **SearchTemplates.xaml** — Removed "SEARCH CASCADE" temporary UI block from the Image block settings gear menu.
- **CaptureOverlay.xaml / CaptureOverlay.xaml.cs** — Fixed compiler error `MC3072` by replacing the invalid native `MouseDoubleClick` XAML handler on `Border` with an internal `ClickCount == 2` check inside the left button down event.

## 2026-07-16 — Removed Image Source Window Label

- **SearchTemplates.xaml** — Removed the `SearchImageSourceLabel` text block showing the captured app name and window title on the image block row.
- **MacroItem.cs** — Removed the unused `SearchImageSourceLabel` property and cleaned up its notifications.
- **SearchTemplates.xaml (ImagePanelInlineTemplate)** — Restored the margin on the `StepName` text block back to `0,0,15,0` (it was previously reduced to `0,0,4,0` to make space for the now-deleted source label) to align it perfectly with the other macro block rows.

## 2026-07-16 — Added Skip Search Area Prompt Setting

- **AppConfig.cs** — Removed `[JsonIgnore]` from `AutoFlowImageCapture` to allow serialization to settings configuration.
- **MacroEditorViewModel.Capture.cs** — Updated `CaptureImageAsync` flow to always show the Image Studio, and then check `AutoFlowImageCapture` (now named "Skip Search Area Prompt"); if disabled (false), it automatically triggers `CaptureScopeAsync` right after.
- **SettingsDashboardView.xaml / SettingsView.xaml** — Restored and renamed the setting in both the Global Settings dashboard and the Inline Settings popup to "Skip Search Area Prompt", showing it under a renamed "IMAGE & COLOR CAPTURE" dashboard card.

## 2026-07-16 — Added Right-Click to Cancel Capture Overlay

- **CaptureOverlay.xaml.cs** — Added a right-click mouse event check inside the capture overlay window's `Window_MouseDown` event handler, allowing users to exit/cancel the capture session by right-clicking.

## 2026-07-18 — Removed Tip Jar & Donation References

- **MainWindow.xaml / MainWindow.xaml.cs** — Removed `TipJarPopup` border XAML element and its code-behind logic (`CheckTipJarStatus`, `DismissTipJarPopup`, click handlers).
- **SettingsDashboardView.xaml / SettingsDashboardView.xaml.cs** — Removed the support developer card content control and its browser open link handler. Updated the stats reset checkbox tooltip to remove the mention of tip jar triggers.
- **SettingsDashboardViewModel.cs / MainViewModel.cs** — Removed the `ScrollToTipJarRequested` static event, and the `HasUnseenTipJarNotification` property and logic.
- **AppConfig.cs / ConfigManager.cs** — Wiped out setting variables related to the tip jar popup (`HasTipped`, `HasDismissedTipPopup`, `TipPopupShownCount`) from loading, saving, and reset stats logic.

## 2026-07-18 — Smart Box Cascade Bugfixes, Simple Capture, and Search Area Overlays

- **ScriptCompilerService.SingleStep.cs & ScriptCompilerService.cs** — Fixed coordinate rebasing logic for `WIN_SMART:` scopes. Included scope parsing adjustments in both the preview and macro execution compilers to ensure dynamic offset rebasing correctly handles window position movements.
- **MacroEditorViewModel.Capture.cs** — Resolved process-name collision bug when multiple window instances are open. Prioritized direct window coordinate retrieval from the overlay's detected handle over the process list lookup.
- **MacroEditorViewModel.Capture.cs (Simple Capture)** — Moved the `UseSimpleCapture` early-exit to run after source window detection. This retains target window metadata (allowing Window Fallback recovery when moved) while keeping the capture workflow simple.
- **ScriptCompilerService.SingleStep.cs (Search Area Overlays)** — Injected a cyan bounding box debug highlight to display the Smart Box search area on screen during rebased matches in preview mode.
- **SettingsDashboardView.xaml** — Unified the "Last Position Cache" settings card layout with standard premium styled rows and simplified its text description.
- **Obsidian Vault** — Added a comprehensive `Image Search Cascade and Smart Box Preview.md` guide explaining cascade stages, window rebasing flow, and visual debugging highlight color maps.

## 2026-07-20 — Trial Extension Codes & Security Hardening

- **Supabase Database** — Created `trial_extension_codes` and `trial_code_redemptions` tables. Developed and optimized `redeem_trial_extension_code` stored procedure with user validation, target calculation from registration date, and campaign/multi-use limits. Added `failed_redemption_attempts` table to log and enforce brute-force protection (lockout after 5 failed attempts in 15 minutes). Enabled Row-Level Security (RLS) on all new tables with corresponding access policies.
- **Admin Portal (Vercel Website)** — Upgraded `private.html` with dropdown fields to toggle between Premium (Dodo) and Trial (Supabase) codes. Switched trial creation inputs and labels to target "Total Trial Days" (e.g. 30 days) instead of "Days to Add". Enabled custom "Redemption Limit (Uses)" inputs for trial codes in single and bulk generators.
- **PowerX Keys App (WPF)** — Integrated a new "Redeem Promo Code" card inside the Settings ➔ Account panel. Wired up TextBox, Redeem button, status feedback messages, and real-time state refresh to the underlying database RPC via `SettingsDashboardViewModel.cs`.

## 2026-07-21 — Auth & Promo Code Audits

- **SupabaseAuthService.cs & SettingsDashboardViewModel.cs** — Fixed PGRST202 parameter mismatch (prepended `p_` to database RPC arguments) and solved JSONException crash by parsing `result.Content` instead of `result.ToString()`.
- **SettingsDashboardView.xaml** — Dynamically bound the promo code card visibility to `IsOnTrial` so the section collapses automatically for paid users.
- **SettingsDashboardViewModel.Commands.cs** — Resolved account switch synchronization bug. When logging out and signing in with a new user, the app now queries fresh subscription data, resets cached status values locally, saves the session, and triggers property change notifications to dynamically update the UI email and tier info.
- **Supabase Database** — Simplified status and error messages returned by `redeem_trial_extension_code` function (e.g. "Invalid code.", "Code limit reached.").
- **Packaging Prep** — Documented requirement to run a deep scan using a powerful model (e.g. Gemini 1.5 Pro) to audit the entire authentication, OTP, and promo system before initiating final packaging.

## 2026-07-21 — Preview Execution & Debug Logging Improvements

- **AhkErrorSuppressor.cs** — Removed `test_preview_step.ahk` from automated window suppression so single-step preview scripts complete their full 2.5-second highlight sleep without premature process termination.
- **ScriptCompilerService.SingleStep.cs** — Fixed `centerX`/`centerY` variable scope calculations across Window Fallback and Screen Fallback search blocks to prevent script errors. Cleaned process name formatting in `_SearchSmartBox_` and `_SearchProcessWindows_` functions.
- **MacroEditorViewModel.Capture.cs** — Added `[PREVIEW_ERROR]` log recording via `DebugLogger.Log` so all preview AHK error tracebacks write directly to `debug_log.txt` while showing clean user-facing warning popups.

## 2026-07-22 — Smart Box & Window Fallback Candidate Selection Fix

- **ScriptCompilerService.SingleStep.cs** — Updated `_SearchSmartBox_` and `_SearchProcessWindows_` helper functions to prioritize the recorded target window title (`SearchImageSourceWindowTitle`) and active window before iterating other process windows. Fixed AHK v2 statement syntax inside candidate loop blocks.

## 2026-07-22 — Pixel Search Precision & AHK v2 RGB Order Fix

- **ScriptCompilerService.cs & ScriptCompilerService.SingleStep.cs & OffsetCaptureWindow.xaml.cs** — Resolved 100% failure rate on single-pixel captures caused by BGR color format conversion (`0xBBGGRR`). Reverted to standard **RGB (`0xRRGGBB`)** to match AHK v2 `PixelSearch` native format.
- **CaptureOverlay.xaml.cs** — Enforced `NearestNeighbor` interpolation and `PixelOffsetMode.Half` on screenshot crops in `SaveScreenshot` to eliminate GDI+ color blurring on thin lines and 1-pixel targets.
- **ScriptCompilerService.SingleStep.cs** — Removed pre-search cyan scope GUI overlay (`_scopeGui`) that was painting over single-pixel search coordinates prior to `PixelSearch` execution. Added target color, tolerance, and 3x3 surrounding live screen pixel color diagnostics to `debug_log.txt`.


