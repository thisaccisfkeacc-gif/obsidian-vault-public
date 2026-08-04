# 🧩 App Modularization — Speed Up Development

## The Problem
The app is getting bigger and more complex. Every change rebuilds everything. Bug fixing takes longer because verifying, fixing, and rebuilding all happen in one big project. AI agents also take more time to navigate and understand the codebase.

## Current State
Everything lives in **one single project** — Models, Views, ViewModels, Services, Templates, Compiler — all in one place. Any small change triggers a full rebuild.

## Proposed Solution: Split Into Separate Projects

### Project 1: PowerX.Core
- Models (MacroItem, MacroStep, AppConfig)
- Database (MacroDatabase)
- Enums and shared types
- **Changes rarely** — most stable part

### Project 2: PowerX.Services
- ScriptCompilerService (AHK generation)
- MacroExecutionService (C# preview engine)
- StopService, ConfigManager
- **Changes when fixing execution bugs**

### Project 3: PowerX.UI
- Views (MacroEditorView, SettingsDashboard, etc.)
- Templates (KeyboardInputTemplates, SearchTemplates, MiscTemplates, etc.)
- ViewModels
- Styles and converters
- **Changes most often** — UI tweaks, compact blocks, etc.

### Project 4: PowerX.App (Main)
- App.xaml, MainWindow
- Startup logic, dependency wiring
- References the other 3 projects
- **Changes rarely**

## Benefits

### 1. Faster Builds
- Change a template? Only PowerX.UI rebuilds
- Fix a compiler bug? Only PowerX.Services rebuilds
- Touch a model? Only PowerX.Core + dependents rebuild
- Currently: EVERYTHING rebuilds every time

### 2. Faster AI Agent Work
- Agent working on UI doesn't need to scan compiler code
- Agent fixing a bug can focus on one project
- Smaller scope = faster understanding = fewer mistakes

### 3. Cleaner Code
- Forces clear boundaries between layers
- No accidental mixing of UI logic with execution logic
- Easier to find things

### 4. Parallel Work
- Two agents can work on different projects without conflicting
- One agent on UI, another on Services — no file conflicts

### 5. Better Testing
- Can write unit tests for PowerX.Services without touching UI
- Can test compiler logic independently

## Other Speed Improvements

### XAML Hot Reload
- For UI-only changes, see results instantly without rebuilding
- Already built into Visual Studio — just needs to be enabled

### Automated Tests
- Write tests for critical paths (compiler output, execution logic)
- Agents run tests instead of manual clicking
- Catches regressions instantly

### Incremental Compilation
- .NET already supports this but it works better with multiple smaller projects
- One big project = slower incremental builds

## Risks & Considerations
- **Big refactor** — needs careful planning, could break things temporarily
- **Circular dependencies** — must be avoided (Core → Services → UI, never backwards)
- **Should be done in phases** — don't try to split everything at once
- **Start with Core** — extract models first (safest, most independent)

## Suggested Phases
1. **Phase 1:** Extract PowerX.Core (Models, Database, Enums)
2. **Phase 2:** Extract PowerX.Services (Compiler, Execution, Config)
3. **Phase 3:** Remaining stays as PowerX.UI + PowerX.App
4. **Phase 4:** Add unit test project for Services

## Questions for Discussion
- Is the codebase ready for this kind of split?
- Are there circular dependencies that would make it hard?
- Should we do this now or wait until current features stabilize?
- What's the estimated effort for Phase 1?

---

## Progress Tracker

**Overall Status:** ~90% Complete (`[█████████░]`)
**Current Stage:** Phase 3 done — UI + App split out; only UI namespace cleanup and Phase 4 (tests) remain

### ✅ Done

**PowerX.Core** (13 files) — Models (MacroItem, AppConfig, AppEnums, AppConstants, ActionModels, AIChatMessage, ObservableRangeCollection, ViewModelBase, VersionInfo, UpdateInfo, CaptureLibraryEntry, TemplateItem) + IMacroDatabase.cs. Compiles independently into `PowerX.Core.dll`.

**PowerX.Services** (32 files) — MacroDatabase, MacroTransferManager, ScriptManager, TemplateDatabase, ScriptCompilerService (+ .SingleStep, .Helper), MacroExecutionService, RemoteServerService, HotKeyService, HotkeyCaptureHook, MacroRecordingService, UndoRedoService, ConfigManager, DebugLogger, AutoUpdateService, AhkErrorSuppressor, AIFallbackService, DragComplexityAnalyzer, ElevationHelper, ErrorHelper, FindTextService, NativeWindowHelper, NetworkTimeService, ServicesUIHooks, ShortcutManager, SmoothTraceEngine, StopService, SupabaseAuthService, TelemetryService, UIElementCaptureService, WindowCaptureService. Compiles independently into `PowerX.Services.dll`. References Core only.

**PowerX.UI** (73 files) — All XAML Views, ViewModels, Converters, Templates, Styles, plus UI-shell services (TrayIconManager, ThemeService, AlwaysOnTopOverlayService, EasterEggService, ServicesUIHooksUI). Compiles into `PowerX.UI.dll`. References Core + Services.

**PowerX.App (PowerX_Keys_V2)** — Slim exe: App.xaml, MainWindow, AssemblyInfo, FileAssociationService. No longer contains UI or service logic — only startup & wiring.

### ⏳ Remaining (small)

**PowerX.UI** — Final namespace cleanup and consistency pass across the 73 files.

**PowerX.App (PowerX_Keys_V2)** — Thin out last few classes (e.g. FileAssociationService → decide final home).

**Phase 4** — Unit test project for PowerX.Services (never started). This is the only planned phase with zero progress.

### Notes
- Actual layout is **5 projects**, not 4: Core, Services, UI, PowerX_Keys_V2 (exe), PowerX_Updater.
- No test project exists anywhere in the repo as of this update.
- A leftover WPF temp artifact (`PowerX.UI_3cjzbmw3_wpftmp.csproj`) sits in the PowerX.UI folder — harmless, safe to delete.

---

## ✅ Decision (Aug 4, 2026): How to Reach "100%"

The physical split is effectively done. Remaining work is polish + new work. Ordered by priority:

### 🟢 Do soon (low risk, real value)
1. **UI namespace cleanup** — one coding pass across PowerX.UI (73 files) to make the split feel finished.
2. **FileAssociationService** — decide its final home (currently the only odd class left in PowerX_Keys_V2 exe).

### 🟡 Do later (real new work, not cleanup)
3. **Phase 4 — Unit tests** — a brand-new test project for PowerX.Services (compiler output, execution logic). Treat as a **separate future task**, not something to block on during active feature work. Needs careful setup and dedicated time.

### ⚪ Low priority / ignore
4. **Delete `PowerX.UI_3cjzbmw3_wpftmp.csproj`** — harmless leftover; delete whenever convenient, not urgent.
5. Don't obsess over a mathematical "100%" — heavy lifting is finished; the rest is polishing.

**⚠️ No consensus reached on:**
- Whether to do the namespace cleanup immediately or after current feature work stabilizes. **Default: proceed with cleanup on next available session.**

**Last Updated:** August 4, 2026
