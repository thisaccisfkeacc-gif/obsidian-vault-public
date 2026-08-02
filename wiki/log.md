# 📝 PowerX Keys Log

---

## 2026-08-02
- **Conflict & Safety System Deep Scan — 6 bugs found & fixed** (see `status/known-issues.md` Bugs 100-105):
  - Key capture case mismatch, name-vs-ID duplicate checks, case-sensitive conflict grouping, empty "Include app" scope loophole, disabled cards blocking key reuse, dead `PressAndRelease` trigger removed (data-safe).
- **AI Assistant**:
  - Anti-spam guard added (progressive 3s → 10s → 30s cooldown, auto-reset after 60s, spam burns daily quota).
  - Free daily limit raised 15 → 30 (client-side default; **server-side Free 30 / Paid 80 pending in Supabase Edge Function**).
- **"None" cards restart cleanup fixed**: startup cleanup now only deletes truly empty cards (no key, no trigger, no image/pixel/app) instead of all unassigned ones.
- **UI Element Capture — Element Picking & Tree Navigation Overhaul** (`PowerX.Services/Services/UIElementCaptureService.cs`):
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
