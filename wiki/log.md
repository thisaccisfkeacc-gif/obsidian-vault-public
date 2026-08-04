## 2026-08-04 — Desktop mouse actions no longer fail window lookup (drag/click on the Desktop)
- **Problem:** Recording a file drag/click on the Windows Desktop stored a target window title ("Desktop") and used window-relative coordinates. At compile/preview the script did `WinExist("Desktop")`/`WinActivate("Desktop")` — no window is named "Desktop", so lookup failed ("Window not found") or the drag landed relative to the wrong window.
- **Recording fix** (`MacroRecordingService.cs`): new `IsActiveWindowDesktop()` guard on all 4 window-coordinate blocks (click, drag, trace×2). When the active window is the Desktop, mouse actions record as plain **Screen** coordinates (the Desktop spans the whole screen, so absolute coords are correct).
- **Main compiler fix** (`ScriptCompilerService.cs`): new `IsDesktopCoordinateTitle()`; MouseClick and MouseTrace compile paths force `coordMode="Screen"` when the coordinate title is Desktop-like — defensive fallback so even pre-existing recordings with `CoordinateMode="Window"`+`"Desktop"` play back correctly instead of emitting the failing lookup.
- **Preview fix** (`ScriptCompilerService.SingleStep.cs`): WindowAction preview now detects `IsDesktopWindow`/Desktop title and shows a success confirmation instead of `WinExist("Desktop")` → "Window NOT found".
- Build verified: 0 errors.
- Note: Task #8 Move Up/Down fix is verified-correct in code + empirically (test harness), pending user re-test of the fresh build.

---

## 2026-08-04 — Recording widget: double-click resets position + expired screen fixes
- **Expired screen polish:** `SubscriptionExpiredWindow` was forced always-on-top (`Topmost` in XAML + `App.xaml.cs`) with no drag support. Removed the Topmost settings, added mouse-drag to move it; replaced the purple inner border gradient (`#3D2B70→#1E1F2A`) and separator (`#2A1D55`) with neutral grays (`#1E1F2A`, `#28293A`) matching the app palette.
- **Double-click reset on floating stop button:** `RecordingWidgetView.xaml` wired `PreviewMouseLeftButtonDown`; code-behind tracks click times and a second click within 300ms (anywhere except the stop button) calls `ResetPosition()` and instantly moves the widget back to its default spot (bottom-right of the working area). `ResetPosition()` (also used by the Settings reset button + settings double-click) now moves the open widget via a shared `MoveToDefaultPosition()` helper instead of only clearing the saved position; the Loaded handler reuses the same helper. No build ran (per user instruction) — pending user test.

---

## 2026-08-04 — Recording fixes (Task #2 & #3 from audit): widget filtering + ghost drag cleanup
- **Task #2 — Floating widget no longer recorded at the source.** `record_engine.ahk` now filters every mouse event (L/R/M button, wheel, XButton1/2, and the TrackMouse move stream) by the window handle under the cursor via new `IsPowerXWindow()` — the floating widget is a child window of the PowerX process, so clicking/dragging/wheeling it never even reaches the C# recorder. Filter is by "PowerX" process name only (NOT "AutoHotkey", so users can still record against other AHK-based windows). Pre-existing C# widget filter kept as a second layer.
- **Task #3 — Ghost "Click & Drag" + trailing Wait removed.** In `MacroRecordingService.MouseUp`, a drag in No-Trace mode used to emit the full path trace AND a second minimal "Curved Drag Trace" that was missing EndX/EndY/absolute-end coords and wrote its trace to the wrong folder (`BaseDirectory\Traces`, pipe format). `FlushActiveTrace` now returns whether it actually emitted a trace; the minimal fallback only runs when the full trace did NOT, and it now carries X/Y/EndX/EndY/AbsoluteX/Y/AbsoluteEndX/EndY and writes a proper CSV `trace_{id}.dat` into `AppConstants.TraceDataFolder`. `MacroEditorViewModel.Recording.cs` adds a final stop-sanitize pass inside `cleanAndOptimize` that strips any surviving tail junk (delays + MouseTrace steps with no end coordinates).
- Changed: `Scripts/record_engine.ahk`, `MacroRecordingService.cs`, `MacroEditorViewModel.Recording.cs`. Build verified: 0 errors.

---

## 2026-08-04 — Delay controls merged into one dropdown (UX simplification)
- Old editor toolbar had two cramped icons: **"Hide Delay Blocks"** (stopwatch E916, ToggleButton bound to `IsDelayHidden`) and **"Auto Smart Trim"** (reload E895, Button). The stopwatch toggle was missing the 4px right margin its neighbors used, so the pair rendered jammed together (user-reported overlap).
- Now a **single stopwatch button** ("Delay options") opens a context menu with two entries: **"Show / Hide Delay Blocks"** (checkable, `IsChecked` TwoWay bound to `IsDelayHidden`) and **"Auto Trim Long Delays"** (runs existing `AutoTrimDelaysCommand` with its confirm dialog). The active "delays hidden" state still shows on the button (blue + slash overlay) via a `DataTrigger` on `IsDelayHidden`; 4px right margin restored (`Margin="0,0,4,0"`).
- Changed: `MacroEditorView.xaml` (replaced both controls), `MacroEditorView.Events.cs` (added `DelayOptionsButton_Click` to open the menu at the button). Build 0 errors (one retry needed — transient obj-dir lock from a killed MSBuild temp project, not a code issue).
- UX is a live draft: user will decide whether to keep the dropdown or revert after seeing it in the running app.

---

## 2026-08-03 — Easter Egg system health check & polish (quality pass)
- **NameDropper — canonical meaning consolidated** (was conflicting in 3 places): trigger = **5 fast clicks on the star filter** (MainViewModel.cs:1154, `ToggleFilterCommand`); identity = "Name Dropper" with a new **"M" monogram glyph** (was duplicate star path), rose accent (TokenRose300), rarity pill "MYTHIC ROSE" (was "GOLDEN STAR • 1 / 6 SECRETS" — hardcoded-count bug). Title "Star Collector"→"Name Dropper", flavor rewritten, `AppConfig.cs` enum comment fixed (`#4 Name macro "Maaz"` → actual trigger), MainViewModel comment fixed. All 6 glyphs now unique (star = VersionBadge only).
- **Pill vs counter**: pill shows rarity only; counter uses real `EasterEggService.UnlockedCount` (verified live: "1 / 6 Secrets Discovered" after fresh unlock).
- **Thread-safety**: `EasterEggService.TryUnlock` now guarded by a static lock — two near-simultaneous triggers (e.g. Konami + "powerx") can't double-unlock or double-popup.
- **Drag fix**: `Window_MouseDown` previously checked `OriginalSource is not Button` — clicks on a Button's template TextBlock (✕, primary) started a drag instead. Now walks the visual tree (`FindAncestor<Button>`), so buttons never drag, empty areas still do.
- **Newly unlocked pip pulses** (opacity pulse, forever-loop, cleared on re-set) so the just-found pip stands out from the rest.
- **Focus/a11y**: primary button named `PrimaryButton` and focused on open; rarity pill 9.5→11px, discovery line 9.5→11px + brighter (#9CA3AF).
- **Removed dead `ProgressBarWidth`** (bar was replaced by pips long ago; nothing referenced it).
- **Verified clean**: sound on UI thread with fallback chain tada.wav → notify.wav → SystemSounds.Asterisk, disposed on close, not double-played; confetti tick-capped at 45 (~90 particles), removed on animation complete, timer stopped in `OnClosed`; Esc + ✕ close; no Copy button / MessageBox present (feature already removed — nothing to fix); animations are RenderTransform/Opacity only; brushes pre-frozen.
- **Live test**: launched app, triggered "powerx" → popup opened with correct title/flavor/pill/counter (verified via UI Automation text dump), Esc closed it. Build 0 errors.
- **Notes for future**: (1) real config path is `%LOCALAPPDATA%\PowerXKeys\config.json` — AGENTS.md says `AppConfig.json` (doc mismatch, not fixed). (2) `UnlockedAtUtc` timestamps deliberately NOT added — would require JSON shape change/migration; do it with the future "Secrets Vault" phase. (3) The `catch { }` in `TryUnlockAndShow` silently swallows popup errors — kept as-is (original design), but it made debugging hard; consider logging there later.

---

  - Fix: the skeleton now only turns ON when the rebuild actually takes long enough to need it (`SkeletonOnDelayMs = 120`, `Task.WhenAny(buildTask, Task.Delay(...))`) — fast rebuilds (single-step add/remove ~55ms) never flip the flag, so no flicker with or without Shift. The turn-off path (incl. the post-recording `_minSkeletonHoldUntil` 400ms hold) is unchanged; the recording-stop path still turns the skeleton on directly at stop (Recording.cs:250), so big-macro processing still shows it as intended.
  - Build verified: 0 errors.

## 2026-08-03 — Dashboard card padding + sidebar star spacing (UI alignment)
- **Task 1 — Card padding alignment** (unified inner horizontal padding = 18px, symmetric): `ScriptLibraryView.xaml:700` Quick Action cards `Padding 15,12 → 18,12`, right margin `4 → 0`; `CustomActionCard.xaml:139` root right margin `4 → 0` (inner grids already 18); `TextSnippetsView.xaml:92` snippet card `Padding 20,15 → 18,15`. Net: all three card types now place text 28px from BOTH viewport edges (10 scroll padding + 18 inner) — no more left/right drift between sections.
- **Dashboard star spacing** (`MainWindow.xaml` SAVED MACROS header): the ★ master-filter button was pinned `HorizontalAlignment="Right"` (full-width empty gap next to the label). Moved into a 2-column header Grid with the label (col 0) and the star (col 1, `Margin="8,-5,0,0"`) so it now sits ~13px after the "SAVED MACROS" text. Command + gold-active trigger untouched.
- Build verified: 0 errors.

---
  - Redesigned `IsAnalyticsPopupOpen` popover card in `MainWindow.xaml` to match the Gold Standard Variation 1 prototype (`#16141F` dark surface, 44px hero glass emoji box, single clean shoutout pill, hero & twin stat tiles).
  - Fixed sharp edge HWND popup clipping by adding transparent outer padding container (`Margin="15"`) and strict corner clipping (`ClipToBounds="True"`).
  - Cleaned up visual clutter: removed `SCORECARD` and `Hero KPI` tags, removed muddy opacity gradients, highlighted footer quote in `#FFD166`.
  - Build verified: 0 errors.

- **In-app OAuth (WebView2) review & hardening** (after other agent's option-1 implementation):
  - Reviewed `OAuthWebViewWindow` (embedded WebView2 Google sign-in). Verdict: solid, builds clean.
  - **Race fixed**: window auto-closed 500ms after seeing `access_token=` in the URL (at `/auth/callback`, BEFORE the local listener received the token) → could hang on "Signing in..." forever. Now closes only when the browser reaches `localhost:54321/token` (token already delivered) + 5-minute safety timeout.
  - **Browser fallback added**: if WebView2 init fails, the URL opens in the default browser and the token listener KEEPS running (previously the listener was cancelled → silent failure). Token wait capped at 5 min → clear "timed out" message.
  - Known limitation: email magic-link flow still uses the default browser (email links can't be intercepted); code-entry flow is clean.
- **Quick-tap "Hold" bug — trace-flush part (complements the other agent's final-lists fix)**: `MacroRecordingService` KeyUp branch flushed the active mouse trace BEFORE dispatching "Released Up"; a trace between a key's down/up blocked the live merge → key stayed "Hold". Moved `FlushActiveTrace()` to AFTER the release dispatch. Safe for real holds (drag traces flush at mouse-up, before the key release).
  - Build verified: 0 errors.

- **"Press" instead of "Hold" for quick taps + desktop drag label**:
  - **Quick tap → "Press" (fix)**: Root cause — the live Key Press Auto-Merge in `MacroEditorViewModel.Recording.cs` converted quick taps (Hold Down + Released Up, no delay) to "Press" only in the live `_targetCollection`; the final recorded lists (`_recordedNoTraceSteps` / `_recordedAllTraceSteps`) kept raw "Hold Down"+"Released Up" clones, so at stop the raw pair replaced the merged "Press" and the timeline showed "Hold A". Fix: (1) live merge now scans back past the auto-injected delay (only when `AutoDelayEnabled` and the tail Delay == `AutoDelayMs`), so the merge works with auto-delay ON too; (2) new `ConvertLastHoldToPress` helper converts the trailing matching Hold Down in BOTH recorded lists so the final timeline shows "Press A". `MacroStep` has no Equals/GetHashCode override → reference equality, so this targeted trailing-match is safe.
  - **Smart View gates**: Pass 3 text bundling (`MacroEditorViewModel.SmartView.cs`) and typing-key detection (`MacroEditorViewModel.Core.cs`) were keyed on `KeyActionType == "Hold Down"` only — since taps now record as "Press", typing keys would fall out of text bundling. Updated all gates to accept "Hold Down" OR "Press". Compiler already emits `Send("{comboAhk}")` for "Press" (ScriptCompilerService.cs ~1984).
  - **Desktop drag label (fix)**: Desktop windows (Progman/WorkerW, `explorer.exe` with "Program Manager"/empty title) were labeled "explorer" in recorded steps. New `IsDesktopWindowTitle` helper in `MacroRecordingService.cs` detects them (real Explorer file-manager windows have real titles → stay "explorer"). Desktop is now labeled "Desktop": initial enumeration (line ~402), `EmitWindowBlock` (windowLabel + new `IsDesktopWindow` flag on the step — flag already consumed by compiler desktop detection at ScriptCompilerService.cs:3045). All coord titles (clicks/traces/drags) copy `_activeWindowTitle` so drags from the desktop now show "Desktop" automatically; the Move/resize step path uses it too.
  - Build verified: 0 errors.

---

- **Round-2 polish — AI panel, analytics popup, sidebar (after crash fix)**:
  - **AI Assistant (AIAssistantView.xaml)**: removed outer gradient stroke (now solid elevated panel + clean border), removed BETA badge, removed "Experimental Feature" warning banner, removed requests-remaining pill, removed ASK/BUILD mode chip from input bar (input row now = text field + send button). Header re-arranged: title + status dot inline, Online/Offline text below. Corner radius normalized to 18.
  - **Click-outside-to-close UX**: extended `MainWindow_PreviewMouseDown` — clicking anywhere outside the AI popup (and not on its toggle button, now `x:Name="AIAssistantToggleButton"`) closes it, same pattern as the analytics popup.
  - **Analytics popup (MainWindow.xaml)**: glass background → solid elevated background (no more see-through); added dimming veil over the hero gradient + softer glow spots; shoutout badge enlarged (FontSize 12.5, padding 14,7); icon boxes fixed — opacity was applied to the whole Border (icon included) so icons rendered as dark blobs at ~14% opacity — switched to alpha-tinted `SolidColorBrush` backgrounds so icons render at full brightness (also fixed the dim AUTOMATION chip text); "Snippets Used" cutoff fixed via smaller icon (32) + tighter margins + 9.5pt label; footer quote switched to amber MDL2 lightning glyph for contrast.
  - **Sidebar**: SAVED MACROS header indented 12px to align with nav item text; star/filter button right margin removed; bottom button row right margin 25 → 20 to align with nav pill right edge.
  - Build verified: 0 errors, launch clean (no crash file, no CRASH log entries).

---

- **Crash fix after redesign (same day, user-reported "something went wrong" at startup)**: The AI panel redesign used 5 Color resources (`TokenPurpleBorder` x3, `TokenRedBorder`, `TokenOrangeBorder`) where Brushes were required (BorderBrush/Background). DynamicResource resolves at runtime, so BAML compiles fine but startup threw `'#FF4C2B80' is not a valid value for property 'BorderBrush'`. Replaced with `TokenPurpleBorderBrush` / `TokenRedBorderBrush` / `TokenOrangeBorderBrush`. Lesson: smoke tests must check `crash.txt` + `CRASH:` in debug_log — the error dialog keeps the process alive, so "process running" is a false positive. Verified: no crash file, no new CRASH entries after launch.
  - Build verified: 0 errors, launch clean.

---

## 2026-08-02
- **Full redesign — Performance Analytics popup + AI Assistant panel (checkpoint: pre-analytics-and-ai-redesign)**:
  - Created git checkpoint first (commit `1df760d`) so both UIs could be rebuilt freely; added `*_wpftmp.csproj` to `.gitignore` (stale WPF temp projects were corrupting builds).
  - **Analytics popup** (MainWindow.xaml): replaced flat stat list with an "Aurora Scorecard" — gradient hero banner (glow spots, badge-wrapped tier emoji, spark-burst animation preserved, big time-saved hero number, shoutout pill) + stat tiles (Macros Fired hero tile with AUTOMATION chip, AI Generated / Snippets twin tiles) + motivational footer. All bindings (EstimatedTimeSaved, TimeSavedEmoji, TriggerSparkAnimation, etc.) untouched.
  - **AI Assistant** (AIAssistantView.xaml): full "Neural Panel" rebuild — gradient shell with purple→blue border glow, glowing AI orb with pulsing ring + spin when busy, Online/Offline status dot, segmented pill mode tabs (purple chat / green build), new user/assistant chat bubbles (AI identity chip, green glow "Build Macro" button), "PowerX AI is thinking" typing indicator, requests-remaining pill (LimitText now actually shown), mode chip + gradient send button with glow in the input bar. All commands/events preserved (ChatScroller, ChatInput_PreviewKeyDown, MessageTemplateSelector).
  - Build verified: 0 errors, app launches clean.

---

## 2026-08-02
- **Settings Dashboard UI Polish**:
  - Mouse Movement Style dropdown: added fallback and default selection so it is never empty on load.
  - Image & Color Capture card: aligned toggle switch right edges consistently.
  - Verified build clean with 0 errors.
  - **Phase 1 — Smarter element filtering:** Hover no longer highlights empty layout panes (Pane/Group/Custom with no Name and no AutomationId). New `IsMeaningfulControl()` decides what counts as a real control; `FindDeepestChildAtPoint()` now returns `null` when only nameless panes sit under the cursor (previously it returned the pane itself, causing whole-window-ish highlights on empty areas).
  - **Phase 1 — Whole-window fallback:** When no meaningful child is found under a large `Window`, the window element itself is now returned instead of `null` — fixes games, Electron/CEF canvases, and owner-drawn apps being uncapturable.
  - **Phase 2 — Tree navigation:** Up/Down arrow keys step to parent / first child; Ctrl+MouseWheel steps the tree while frozen (wheel up = deeper). Navigation locks the highlight until the cursor moves >4px; the label shows tree level (`🔒 LOCKED (Tree, level N)`). Mouse hook now intercepts `WM_MOUSEWHEEL` while Ctrl is held (reads `MSLLHOOKSTRUCT.mouseData`).
  - ⚠️ **For future agents:** any UI element capture regression (highlight picking, freeze, hover behavior) — suspect this change first. Details + old-vs-new behavior in `wiki/services/ui-element-capture.md` § Element Picking & Tree Navigation (2026-08-02).
  - Build verified: 0 errors.
- **Light Theme Audit & Fix (delegation prompt `.kiro/steering/light_theme_delegation.md` + `Light_Mode_Implementation_Directive.md`)**:
  - Tokens: `TokenBorderDefault` → `#CBD5E1` in LightTheme; new theme keys `GhostKeyOpacity` (Light 0.6 / Dark 0.25), `TokenTabActiveText` (Light #0F172A / Dark #FFFFFF), scrollbar thumb tokens (Light #CBD5E1/#94A3B8/#64748B; Dark unchanged). Key-for-key parity kept between theme files.
  - Keyboard Manager: ghost key opacity now theme-aware via new `GhostKeyOpacity` resource (previously converter fell back to 0.25 in both themes).
  - AI Assistant modal (`AIAssistantView.xaml`): header+input bg unified to panel white (fixes double-border); active tab text now theme-aware (was hardcoded #FFFFFF → invisible in light); inactive tabs darkened to muted; user-bubble/build-button/send-icon text → White; BETA badge, section icons → purple600; added drop-shadow.
  - Macro Editor: toolbar icons (Undo/Redo/toggles/gear) → TokenTextSecondaryBrush; step filter icon → purple600.
  - Script Library: `HoverCardStyle` got a visible `#CBD5E1` border (was Transparent → cards blended into bg); section headers + icons darkened; "Assign" chip border/text strengthened + purple hover; "Listening..." text → purple600.
  - Settings/Account: toggle ON → brand purple (#7C3AED) instead of generic blue; "Upgrade to Premium" button → purple gradient (was orange); scrollbars → soft slate in light; promo icon + zoom badge + version-history toggle darkened.
  - Directive items verified already-fixed: App.xaml `PremiumComboBoxStyle` (3 sub-items), DropdownPromptWindow combo foreground, MacroStepCard delete button.
  - ⚠️ **For future agents:** if a light-theme color looks wrong after 2026-08-02, suspect the token changes first (`TokenBorderDefault`/`TokenTextMuted`/new keys). Audit screenshots at `C:\Users\Maaz\Desktop\Light mode screenshots`.
  - Build verified: 0 errors (232 pre-existing warnings: NU1510/NU1902/NU1903 + nullable/obsolete).
- **Macro Editor — Bulk Context Menu + Undo/Redo Overhaul**:
  - All right-click context-menu actions are now multi-select aware: Move Up/Down, Duplicate, Disable, Human Flow, Group, Move Inside/Outside operate on the whole selection in one undo entry. Bulk menu headers use "Disable 5 Steps" format.
  - `MacroEditorViewModel.Core.cs`: `MoveSelectedSteps`, `RunBulkAction`, `ToggleDisableStepCore`, `ToggleHumanizationStepCore`, `DuplicateSelectedSteps`, `NestStepsInto(IList)`, `UnnestSteps(IList)`, `CanGroupSelection(IList)`; undo-push sites guarded by `_suppressUndoPush` so bulk ops = ONE undo entry. Ctrl+D duplicate routes through `DuplicateStepCommand` (one undo).
  - `UndoRedoService.StepsEqual` rewritten (compares ~60 step fields) — adjacent property edits no longer wrongly coalesce into a single undo entry. New `StatesEqual` wrapper.
  - Undo/Redo now restore the pre-undo selection via `CollectSelectedIds` / `RestoreSelectionByIds` (recursive, `s.Id`).
  - **Undo toast**: Ctrl+Z/Y shows what changed ("Undid: added 3 steps", "Undid: moved steps", "Undid: changed settings") via `ShowUndoRedoToast`.
  - **Honest unsaved indicator**: `MarkSaved()` snapshots the step tree at save points (Save button, macro load, AI macro creation). `IsStateEqualToSaved()` returns true when Undo returns to exactly the last-saved state → `IsDirty = false` (no warning on exit). Brand-new macros never "clean". NO auto-save — saving stays manual only.
  - Build verified: 0 errors.
- **CRASH FIX — Macro section (Script Library) crash on view change**:
  - **Crash:** App closes when navigating Dashboard → Macro section. `crash.txt`: `Cannot find resource named 'SearchTextBoxStyle'` (XAML line 1400/1401).
  - **Root cause:** `CustomActionCard.xaml` (hosted in `ScriptLibraryView.xaml:525`) referenced `{StaticResource SearchTextBoxStyle}` on the Scheduled Time input, but that style is defined ONLY in `CaptureLibraryWindow.xaml` — a different window. WPF StaticResource resolution at template load → unhandled exception → app exit. Other card styles (`PremiumComboBoxStyle`) work because they're in `App.xaml` (app-wide).
  - **Fix:** Replaced the broken style reference with inline token-based styling matching the card's other inputs (TokenElevatedBgBrush bg, TokenBorderMediumBrush border, Padding 8,6). Verified only `SearchTextBoxStyle` leaked; `TabItemStyle`/`LibraryListBoxStyle` are self-contained.
  - ⚠️ **For future agents:** if a view change crashes with a "Cannot find resource named X" error, that style is scoped to a different window — check `App.xaml` for app-wide styles and inline token styling for view-local ones. Build verified: 0 errors.
- **Overlay Helper Consolidation (Phase 3 light cleanup)** — new `PowerX.Services/Services/NativeWindowHelper.cs`:
  - `MakeClickThrough(hwnd, addToolWindow)` — replaces 3 identical Win32 click-through blocks (element picker `WS_EX_TOOLWINDOW`; pinned-window + mouse-path `WS_EX_LAYERED`).
  - `GetDpiScaleAtPoint(x, y)` — replaces 2 byte-for-byte DPI copies (CaptureOverlay + WindowCaptureService).
  - `AlwaysOnTopOverlayWindow.ParseHexColor` replaced with canonical `ColorConverter` (sky-blue fallback kept).
  - ❌ Deliberately NOT merged: the 4 overlay windows themselves (different lifecycles/purposes — audit by third agent confirmed). `OffsetCaptureWindow` / `AlwaysOnTopOverlayService` untouched (different DPI math).
  - ⚠️ **For future agents:** overlay click-through/DPI bugs — suspect `NativeWindowHelper` + these 5 call sites first. `UIElementCaptureService.PositionHighlight` still does NOT DPI-correct `BoundingRectangle` (known issue, logged in `ideas.md`).
  - Build verified: 0 errors.

---

## 2026-07-26
- **Fix Desktop Refresh Flickering on App Launch / Restore**:
  - Replaced `HWND_BROADCAST` in `App.xaml.cs` with a temp-file HWND handoff (`%TEMP%\powerx_hwnd.txt`) and `IsWindow()` verification. Single-instance restoration messages now directly target the first instance's HWND without broadcasting to Windows Explorer.
  - Added registry check in `FileAssociationService.cs` before registering `.pxmacro` file extensions. Skips redundant HKCU key writes and `SHChangeNotify(SHCNE_ASSOCCHANGED)` calls if already registered, eliminating desktop icon cache rebuilds on launch.
- **Settings Dashboard UI Polish**:
  - Fixed Light Mode theme circle swatch fill in `SettingsDashboardView.xaml` (`#F8F9FA` light fill vs `#1E1F29` dark fill) so appearance selection circles are clearly distinguished.
- **Section 3 Task & Audit Cleanup**:
  - Verified and removed completed UIElement routing, preview sonar highlight, and desktop fallback items from `ideas.md`.
- **Dashboard Layout Fix (top gap / bottom clip / uneven right padding)** — layout-only, no color or design changes:
  - `ScriptLibraryView.xaml`: removed stray right margin (`Margin="0,0,10,0"` → none) on the cards panel grid; scroll viewer padding made symmetric (`0,0,10,0` → `10,0,10,0`, My Macros one gets `10,0,10,24` for bottom breathing room); removed the 40px top void and trimmed the 140px bottom void inside the Default/Quick Actions/ComingSoon scroller (`Margin="0,40,0,140"` → `"0,0,0,40"`).
  - `TextSnippetsView.xaml`: same right-margin removal (`0,0,10,30` → `0,0,0,30`) + symmetric scroll padding with bottom room (`0,0,6,0` → `10,0,10,24`).
  - Covers My Macros, Default, Quick Actions, Coming Soon, and Text Snippets. `MacroEditorView.xaml` untouched; `MainWindow.xaml` ContentControl margin untouched.
- Build verified: 0 errors.
- **Performance Stats over-saturation (MainWindow.xaml Performance Analytics popup)** — reduced heavy purple to match theme: card border `TokenPurpleBorder → TokenBorderDefault`, glow shadow `TokenPurple500 → #000` 0.35; header divider + hero emoji box borders → `TokenBorderDefault`; hero emoji glow `TokenPurple600 → #000` 0.25; "TOTAL TIME SAVED" caption `300 → 500`; shoutout pill bg `PurpleBgHover → MenuHover`, border neutral, text `Purple100 → 300`; stat icon boxes `PurpleBgSubtle → MenuHover` with calmer icon fills (300→500); snippet tile + footer replaced hardcoded purple-tinted hex (`#1A172B/#2A243D/#181427/#251F3B/#1E293B`) with theme tokens (SurfaceBg/AppBg/BorderDefault/ElevatedBg) — fixes Light mode too. Purple kept only as subtle accents (header tint, small icon fills).
- Build verified: 0 errors.

---

## 2026-07-31
- **Macro Templates Feature — Audited & Completed**:
  - Full end-to-end audit of the Use Template (InsertTemplate) block: storage, UI, compiler, runtime, preview, validation.
  - **Critical fix**: InsertTemplate (45) was a silent no-op in `ScriptCompilerService.cs` + `MacroExecutionService.cs` — now inlines/runs template steps (CallMacro pattern) with recursion protection.
  - Fixed 2 data-loss bugs: Save button in template mode overwrote the host macro; leaving the editor mid-edit destroyed the macro backup.
  - Fixed dropdown state restore, emoji name mismatch, IsValid gap, step-name collisions; deleted `SearchTemplates.xaml.temp`.
  - Build verified: 0 errors.
  - Report: `reports/template-feature-completion-report.md`.

---

## 2026-08-01
- **Turbo Engine Mode — replaced "High Priority Mode" (aka Zero-Latency Override)**:
  - Idea review → smart-boost design (multi-agent input merged): setting renamed `EnableZeroLatencyMode` → `EnableTurboEngineMode` (AppConfig.cs:138), default OFF.
  - Old C#-side "always High" logic removed (ScriptManager.cs start-boost + `UpdateLiveEnginePriority` deleted).
  - Compiler now injects AHK-side `TurboBoost()` per step (sets High + resets rolling 3s decay via `SetTimer(TurboDecay, -3000)`), compiled only when toggle ON — ScriptCompilerService.cs.
  - Toggle change → `Save()` auto-reloads engine. UI renamed with ⚡ icon + new copy (SettingsDashboardView.xaml, SettingsDashboardViewModel.cs, AIAssistant_KnowledgeBase.txt).
  - Build verified: 0 errors.
- **Full Wiki Freshness Audit (4 parallel sub-agents, 118 files)**:
  - ~40 docs updated to match current code: models (app-config, macro-item, database-schema), guides (agent-onboarding, debug-log, adding-a-feature, packaging, win32-gotchas, Multi-AI), features (settings-dashboard, macro-editor, script-library, capture-library, schedule-trigger, mobile-remote, onboarding), services (auto-update, ai-assistant, script-compiler, telemetry, file-search→removed), architecture/managers (script-manager 6 slots, ai-login-manager→removed, Payment System→Supabase-only, component-relationships, overview v5.4.0), audits/bugs/status (audit-tracker, hidden-and-abandoned, known-issues, deep-scan-audit, image-preview-coordinate-bug→fixed, ahk-suppressor, exp-block-studio, theme-color), ADR-0002.
  - Removed-feature notes: Task Scheduler backend, Everything file search (SmartMenuMockup), Tip Jar/Buy Me a Coffee, Visual Keyboard, App UI Zoom, Performance Mode, AI model selector.
  - ux-naming-audit item #3 marked ✅ Resolved (Turbo Engine Mode).
  - Known correct (not "fixed"): traces are `.dat` in `%APPDATA%\PowerX_Keys\TraceData\` (MacroRecordingService.cs:139).
  - Graphify graph refreshed (24991 nodes).

---

## 2026-08-02
- **Tray Macro Focus Restore (fixed, verified working)**:
  - Symptom: clicking a favorite macro in the tray menu executed the macro, but keystrokes did NOT land in the last active window (e.g., Notepad). Editor preview worked because it minimizes the app window, which releases the foreground lock.
  - Root causes: (1) Windows 11 gives foreground to the taskbar when the tray icon is clicked, so GetForegroundWindow() at click time could return the wrong HWND; (2) SetForegroundWindow alone silently fails against the menu foreground lock.
  - Fix in PowerX.UI/TrayIconManager.cs: continuous _focusSampler timer (every 400ms) records the last *usable* target (excludes own process, Shell_TrayWnd, Progman, WorkerW, menu windows); IsUsableTarget() filter; ForceActivateWindow() (Alt-tap + BringWindowToTop + SetForegroundWindow + SetFocus, then AttachThreadInput fallback); 5-attempt verify loop; last-resort minimize/restore of the target (same mechanism the preview uses). [Tray] diagnostics lines added to debug_log.
  - For future agents: this is the model for any "restore focus after context menu" problem — capture early via sampler, verify with GetForegroundWindow, fall back to minimize/restore.
- **Tray Quit hover blend fix**: RED_HOVER fallback was (42,22,22) — nearly identical to menu BG (24,25,30), making the Quit item unreadable on hover. Changed fallback to (110,30,30); OnRenderItemText now renders white text when a red item is selected.
- **Dashboard padding/gap fix** (ScriptLibraryView.xaml): removed two opaque "Premium Fade Overlay" borders that hid the top 40px and clipped the bottom 60px of the macro grid; trimmed header margin (30→12) and scroll padding (24→10); added HorizontalContentAlignment="Stretch" to 8 ItemsControls so cards span full width.
- **Build-blocking XAML fix** (SettingsDashboardView.xaml): SettingsScrollFadeMask LinearGradientBrush was pasted OUTSIDE UserControl.Resources (MC3023). Moved inside the Resources block. Not ours — was introduced by a concurrent edit; fixed silently.
- Build verified: 0 errors.

## 2026-08-02 (cont.)
- **Recording-stop lag fix (big blocks)** — verified real, fixed:
  - Symptom: after stopping a recording, the skeleton loader stays visible / stutters, noticeably on large complex blocks.
  - Root cause: the stop handler ran ALL heavy work on the UI thread while the skeleton was up — (1) DumpStepsToLog wrote every recorded step to %DOCUMENTS%/PowerX_Keys/recorder_debug.log (file I/O per stop), (2) DetectLoopPattern was O(n^2) over the whole timeline, (3) two full optimize + window-collapse passes.
  - Fix in MacroEditorViewModel.Recording.cs + Optimization.cs: heavy cleanup/optimization moved into a background Task.Run (all pure list ops on non-UI-bound lists; UI thread only does the final insert + RefreshDisplaySteps); DetectLoopPattern now skips timelines > 500 steps (the Repeat-block hint is pointless on big recordings); DumpStepsToLog runs fire-and-forget off the UI thread.
  - For future agents: cleanAndOptimize's captured locals (stoppedByWidgetClick, _cachedWidgetBounds, pendingDelay) are still safe — only plain lists are touched off-thread; keep the final _targetCollection mutation + toasts on the dispatcher.
  - Build verified: 0 errors.

## 2026-08-02 (cont.)
- **Record-button start delay fix (verified via sub-agent audit)**:
  - Audit (explore sub-agent) traced click -> widget: synchronous engine shutdown (incl. Thread.Sleep(200) at ScriptManager.cs:468) ran BEFORE the widget BeginInvoke; AHK recorder cold start 0.2-3s (Defender); ~50 synchronous File.AppendAllText debug writes in 134ms from WinEventHook flood at start (DebugLogger.cs:76).
  - Fix 1 (MacroEditorViewModel.Recording.cs StartRecording): ScriptManager.StopAllProcesses() moved into a background Task.Run — widget appears instantly, engine stops unseen.
  - Fix 2 (MacroRecordingService.cs): ThrottledWinMoveLog() gate (150ms) on the 3 noisy WinMoveEventCallback debug lines — kills the start stutter. _lastWinMoveLogTicks field added.
  - Decided against: boot-time engine pre-warm (File.Copy only happens once — negligible), hover pre-warmup and 24/7 recorder (rejected — overhead/risk for ~200ms gain).
  - Build verified: 0 errors.

## 2026-08-02 (cont.)
- **Honest Record-start UX (vanish only when the engine is actually live)**:
  - Problem (user-identified): the app vanished to the tray the instant Record was clicked, so the user assumed recording had started and began pressing keys — the first keystrokes were lost while the AHK recorder booted (0.2-3s). The vanish was a lie.
  - Fix: the app now stays visible with a pulsing dot + "STARTING..." label (new IsRecorderStarting / RecordButtonLabel on MacroEditorViewModel.Properties.cs, Record button TextBlock now binds RecordButtonLabel). The vanish + floating widget moved OUT of the early BeginInvoke into a new onRecorderReady callback (3rd arg of MacroRecordingService.StartRecording). The service invokes it on the UI thread via Dispatcher.Invoke right after Process.Start + BeginOutputReadLine and BEFORE the initial window snapshot — so the snapshot always sees the user's real target window, never the PowerX window.
  - Safety net (user's method 1): cleanAndOptimize now strips any WindowAction step whose WindowTitle contains "powerx" (matches the recorder's own exe-name filter), so an accidental self-capture can never survive a recording.
  - Error paths all reset IsRecorderStarting (AHK-missing toast path + outer catch), so the button never sticks on STARTING.
  - For future agents: ready-callback ordering matters — vanish BEFORE enumeration, or the first "Window Focus" step points at PowerX. The step delegate, ready delegate and stoppedByWidgetClick all share StartRecording's scope; _floatingWidget is a VM field.
  - Build verified: 0 errors.

## 2026-08-02 (cont.)
- **Stop-lag round 2 — verified remaining UI-thread O(n²) work, fixed**:
  - User asked to verify whether the recording-stop skeleton lag is truly gone. Code audit found two remaining certain bottlenecks on the UI thread at stop:
    (1) Phase-2 merge (MacroEditorViewModel.Recording.cs): per-step _targetCollection.Remove(step) (linear Equals scan each — O(n²)) and survivedSteps = _targetCollection.Where(currentSessionSteps.Contains) (O(n²)). MacroStep has NO Equals/GetHashCode override, so reference equality — HashSet swap is semantics-safe.
    (2) RefreshDisplaySteps (MacroEditorViewModel.Properties.cs): the ENTIRE display rebuild (CreateSnapshot + BuildSmartSteps in Smart Mode — clones and groups every step) ran INSIDE the UI-thread dispatcher callback, freezing the skeleton animation on big macros.
  - Fixes: backward single-pass RemoveAt with a HashSet (O(n)); survivedSteps via insertedSet HashSet lookup; RefreshDisplaySteps now snapshots the model on the UI thread (fast plain-list copy) then does the heavy build OFF-thread, swapping via bulk ReplaceRange on the UI thread. Skeleton now animates during the build.
  - For future agents: keep model reads on the UI thread (snapshot pattern) and only transform plain lists off-thread; keep step-property mutation (RefreshStepFailureStates) on the UI thread.
  - Build verified: 0 errors.

## 2026-08-02 (cont.)
- **Stop-skeleton flicker fix — minimum-hold window**:
  - Verified: the loader turns ON unconditionally at stop, and for tiny recordings the rebuild is so fast (~60ms incl. 50ms debounce) the skeleton flashes on/off — reads as a flicker, not a load.
  - Fix (user-chosen approach, floor not cap): the stop handler sets _minSkeletonHoldUntil = now + 400ms; RefreshDisplaySteps' finally honors it — if now < hold, the turn-off is deferred via a background delay + dispatcher (idempotent: only the first completion clears the hold). Big rebuilds exceed 400ms naturally and turn off immediately, so the hold is invisible to large macros.
  - For future agents: the delayed turn-off compares _minSkeletonHoldUntil == until to stay idempotent across overlapping RefreshDisplaySteps calls (cancelled refreshes also pass through finally). Hold lives only in the stop path — all other RefreshDisplaySteps callers keep instant turn-off.
  - Build verified: 0 errors.
## 2026-08-03 � UX polish batch (12 items, all verified + built clean)
- Desktop dot: Window-title templates now hide the bullet when the title is empty (Desktop focus shows just 'Desktop').
- Multi-select move/delete: verified correct (ordered moves, single undo, keyboard path) � no fix needed.
- Sidebar: nav items left margin 25 ? 15 (less right-push).
- Analytics popup: removed ClipToBounds; header banner rounds its own top corners.
- Recording: selection now always asks 'After Selected Step' vs 'At End of Macro' (abort on Esc), independent of SafetyConfirmations.
- Key Hold/Release mid-recording: verified sound (Press merge + orphan stripping) � no fix needed.
- AI chat placeholder: fixed broken AncestorType binding � now hides on focus via ElementName.
- Settings fade mask: top fade tightened 5% ? 1.5% so headers stay readable.
- Version History: brighter purple text both buttons; ghost key opacity 0.25?0.5 (dark), 0.6?0.75 (light).
- Removed PowerX Intelligence card from Settings; removed Kill Switch Notification toggle (feature stays always-on via AppConfig default true).
- Tray right-click bug: right-click on tray icon no longer touches the main window � menu-only; left/double click and 'Open Window' now always show+activate the window (removed the old F25-W2 toggle that could hide it). Guarded the Win32 foreground trick in ShowContextMenuWithOffset to visible windows only.

- **Performance Analytics Popup Raycast Polish & Corner Verification**:
  - Refined Performance Analytics popup in MainWindow.xaml with Raycast Premium Dark palette (#14121E).
  - Added explicit CornerRadius to Header (16,16,0,0) and Footer (0,0,16,16) borders inside Popup card to guarantee 100% pixel-perfect rounded corners.
  - Highlighted footer quote in bright gold (#FBBF24).
  - Committed changes to git repository.
  - Build verified: 0 errors.

- **Disabled Pre-installed Default Macros for New Installs**:
  - Removed auto-provisioning of sample macros ('Smart Notepad Typer' & 'Stretch Break Timer') in MacroDatabase.cs.
  - New app installations now start with a clean, empty macro library on first launch.
  - Build verified: 0 errors.

- **Multi-Select Move to Top/Bottom (voice_note_feedback_3)**:
  - Added MoveToTop/MoveToBottom EditorAction entries + Ctrl+Shift+Up/Down defaults in ShortcutManager.cs.
  - New MoveStepToTopCommand/MoveStepToBottomCommand (MacroEditorViewModel.Commands.cs) backed by MoveSelectedStepsTo() in Core.cs (order-preserving, grouped by parent container, single undo state).
  - Context menu now shows "Move to Top"/"Move to Bottom" with multi-select bulk awareness (MacroEditorView.xaml + Events.cs); cheat sheet updated.
  - Removed both dead settings earlier this session: Instant Record Mode + Auto Bind Image Target (UI, AppConfig, ViewModels, recording window-hide now unconditional).
  - Closest-to-Cursor UI element option verified already implemented (dropdown + main compiler + runtime) � no code needed.
  - Build verified: 0 errors.

- **Reverted Move to Top/Bottom; fixed multi-select Move Up/Down edge bug (voice_note_feedback_3)**:
  - Removed 'Move to Top'/'Move to Bottom' entirely: enum entries + Ctrl+Shift+Up/Down defaults (ShortcutManager.cs), commands + wiring (MacroEditorViewModel.Commands.cs), MoveSelectedStepsTo method (Core.cs), keyboard + context-menu wiring (MacroEditorView.Events.cs), menu items (MacroEditorView.xaml), cheat sheet rows.
  - Fixed multi-select Move Up/Down: old guards skipped the WHOLE group if selection included the first/last step (Move Up looked dead). Now every selectable step moves individually; only edge-blocked steps stay put.
  - Note: Commands.cs content was overwritten once mid-session by an external process (restored an old '-9999' variant) - re-applied edits; watch for editor/OneDrive overwrites.
  - Build verified: 0 errors.

- **Removed Settings bottom gradient**: deleted 'Premium Bottom Fade Overlay' Border (SettingsDashboardView.xaml). Build verified: 0 errors.
- **Easter Egg redesign doc**: full UI/UX redesign written to Obsidian Vault/ideas/easter-egg-redesign.md (ceremony timeline, unique badge art + rarity per egg, pips progress, Vault gallery + data model, sound/motion, phased rollout). No code changes yet.

- **Easter Egg window themed to app palette (voice task)**:
  - Replaced emoji glyphs with crisp vector Path glyphs per egg (star, heart, lightbulb, M monogram, echo rings, lightning).
  - Accent colors remapped to app tokens: Architect=Amber500 gold, Gamer=Purple400, Curious=Indigo300, NameDropper=Rose300, Whisper=Blue400, ShiftClicker=Green500 (aura, border, pill, glyph all tinted).
  - Fixed rarity pill hardcoded '1/6' bug - pill now shows themed rarity label only; real count stays in counter.
  - Copy Card: no more MessageBox; inline 'Copied' flash on the button label.
  - Drag jank fix: buttons no longer trigger window drag.
  - Confetti recolored to on-palette gold/amber/purple/indigo/blue/white.
  - Build verified: 0 errors.

- **Smart Filter feature (recording persistence)**: new opt-in setting `IsSmartFilter` (AppConfig.cs, default OFF) + VM property (Properties.cs). On recording stop, after CollapseWindowActions, `ApplySmartFilterIfEnabled` (Recording.cs) rebuilds both recorded lists via BuildSmartSteps and permanently replaces the raw stream with the bundled version (real Text steps, Keyboard "Press" combos, ClickCount/ScrollAmount), then flattens by clearing VirtualSourceSteps. Undo snapshot (pre-record, deep-cloned) unaffected; compiler already handles all bundled formats (ConvertComboToAhk etc.). Build verified: 0 errors.

- **Smart Delay Quantization (Smart View)**: new ApplyDelayQuantization pass at the end of BuildSmartSteps() (MacroEditorViewModel.SmartView.cs). Rounds Delay durations to clean values: 50-200ms?nearest 50, 200-1000?nearest 100, 1000-5000?nearest 500, >5000?nearest 1000; <50ms left untouched (dropped upstream). Display-only: pass-through delays referencing raw steps are replaced with rounded Clone() copies (raw CurrentMacro.MacroSteps never mutated) and wired to HandleVirtualDelayChange so manual duration edits still persist. Also feeds the Smart Filter save path via BuildSmartSteps. Build verified: 0 errors.

- **Delay Trim buttons (Auto Smart Trim + Manual Trim)**: C# (Optimization.cs) + XAML (MacroEditorView.xaml) + code-behind (MacroEditorView.Events.cs). Auto Smart Trim (one-click, DarkMessageBoxWindow confirm) computes the macro's median delay rhythm: delays over 3x the median are capped to 3x median; any delay over 8s capped to 3s; under-3s left untouched; runs OptimizeRecordedSteps + recursive adjacent-delay merge afterward. Manual Trim opens an inline popup panel (ToggleButton bound to IsManualTrimPanelOpen) with threshold-seconds + cap-ms inputs, a live 'This will affect N delay steps' preview, and Apply/Cancel. Both push one undo state, mark IsDirty, refresh the display, and toast the result. New handler ManualTrimPopup_Closed resets the toggle when the popup is dismissed by outside clicks. RelayCommand/ICommand, DynamicResource tokens, Segoe MDL2 glyphs throughout. No .ahk touched. Build verified: 0 errors.

- **Wait for Window (auto-detect recording pass)**: new setting `AutoConvertDelayToWaitWindow` (AppConfig.cs, default ON). On recording stop, after CollapseWindowActions and before Smart Filter, `ApplyWaitForWindowIfEnabled` (Recording.cs) scans both recorded lists for [action] ? [Delay] ? [WindowAction Activate] of a window that was not the previously active one, and converts that delay into a WaitUntil step (WaitConditionType=WindowActive, title in Value/WindowTitle, 15s timeout via UITimeoutSeconds). When the setting is OFF, the delay is left untouched but tagged with a transient `SmartSuggestionText` = ""?? Consider converting to Wait for Window"" (new non-persisted property on MacroStep, wired into the existing IsSafetyWarningVisible/SafetyWarningText getters so the delay card badge shows it � no XAML change needed). Compiler untouched (WindowActive + UITimeoutSeconds path already exists). Build verified: 0 errors.

- **App Launch Auto-Detection (recording pass)**: new setting `AutoDetectAppLaunch` (AppConfig.cs, default ON). At StartRecording, MacroRecordingService snapshots every running process name into `BaselineProcessNames` (IReadOnlySet). On recording stop, after CollapseWindowActions and before the WindowAction-only purge, `InsertAppLaunchSteps` (Optimization.cs) + wiring (Recording.cs) scans both recorded lists for WindowAction Activate/Restore blocks, extracts the exe from the `Title ahk_exe name` window title, and inserts a FileLauncher step (Value = resolved MainModule.FileName) before any whose process was not in the baseline. Filters: desktop windows, blank titles, PowerX/AutoHotkey exes, headless processes, one insert per unique exe. Path resolution prefers a visible-window instance, falls back to the exe name when the app already exited. Build verified: 0 errors.

- **Recording Engine & Smart Cleanup Pass (2026-08-03)**:
  - **Smart Filter**: Text bundling & shortcut/chord detection persisted to stored steps via IsSmartFilter setting.
  - **Smart Delay Quantization**: Delays rounded to clean values in BuildSmartSteps().
  - **Smart Mode lightbulb toggle**: Controls both IsSmartMode + IsSmartFilter together (timeline button + settings dashboard synced).
  - **Auto Smart Trim ✂️ button**: Median-based outlier detection, caps long delays.
  - **Manual Trim ⚙️ button**: User-defined threshold + live preview.
  - **Wait for Window auto-detection**: Converts delay→window-switch patterns into WaitUntil (WindowActive) steps, with 💡 hint when setting is OFF.
  - **Launch App auto-detection**: Snapshots processes at recording start, auto-inserts FileLauncher steps for newly launched apps.
  - **Idle Gap Detection 💤**: Visual amber indicator on delay steps ≥ 5 seconds.
- **UIA Element Anchoring (click upgrade suggestions)**: two-piece feature, no setting, always on. (1) Recording pass `EnrichClicksWithUIASuggestions` (MacroEditorViewModel.Recording.cs) � after CollapseWindowActions, silently probes every MouseClick step via AutomationElement.FromPoint(AbsoluteX/AbsoluteY), scoped to the step's CoordinateWindowTitle window + guarded against PowerX's own window covering the spot. Meaningful control-type filter (Button/CheckBox/ComboBox/Edit/MenuItem/RadioButton/Hyperlink/ListItem only). Finder + convert share `TryFindUIElementAt`; badge set via transient SmartSuggestionText (non-persisted). Scroll/Move/Drag and find-target clicks skipped. (2) Right-click - "? Convert to UI Element" in the timeline context menu (visible only on MouseClick steps, MacroEditorView.xaml) - re-probes, populates UIElementName/AutomationId/Class/ControlType/WindowTitle/Path, switches the step to MacroStepType.UIElement keeping X/Y as coordinate fallback, clears the badge, marks dirty. On miss shows a ""Could not detect a UI element at this position"" toast and changes nothing. Mouse card template now renders the SafetyWarningText badge (MouseTemplates.xaml + BooleanToVisibilityConverter resource). Build verified: 0 errors.

## 2026-08-05 - Agent rules sync
- Updated workspace agent rules so Codex, Antigravity, and Kiro stay aligned through .agents/AGENTS.md, .kiro/steering/rules.md, and root AGENTS.md.

