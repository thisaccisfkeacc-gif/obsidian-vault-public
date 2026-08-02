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
