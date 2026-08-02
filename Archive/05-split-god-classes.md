---
tags: [refactor, maintainability, phase-3]
date: 2026-07-23
status: done
---

# 🔧 Fix 05: Split God Classes

**Priority:** 🟡 Medium
**Effort:** 3-4 hours
**Risk:** 🟡 Medium (many files affected)

---

## Problem

Two classes have grown too large and handle too many responsibilities:
1. `ScriptViewModel` — 1300+ lines, handles events, recording, execution, UI state
2. `ScriptEditorViewModel` — 2800+ lines, handles code editing, previews, tabs, undo/redo, AI integration

## Current State

### ScriptViewModel.cs (1300+ lines)
- Event execution logic
- Recording/stop recording
- Script list management
- Hotkey handling
- UI state management
- Multiple async void methods

### ScriptEditorViewModel.cs (2800+ lines)
- Code editor functionality
- Preview management
- Tab management
- Undo/redo system
- AI code generation
- GDI graphics handling
- Multiple nested classes

## Proposed Split

### 3. `ScriptCompilerService.cs` (5,264 lines) — Critical Compiler God Class
- Currently contains all AHK v2 script generation for 30+ block types in a single file.
- **Split Strategy:** Use C# `partial class` files:
  - `ScriptCompilerService.cs` (Core engine & header emit)
  - `ScriptCompilerService.MouseClick.cs` (Mouse & coordinate blocks)
  - `ScriptCompilerService.ImagePixel.cs` (Image search & pixel detection)
  - `ScriptCompilerService.WindowLogic.cs` (Window management & scope control)
  - `ScriptCompilerService.Hotkeys.cs` (Hotkey registration & triggers)

### 4. `MacroExecutionService.cs` (2,801 lines) — Execution Engine
- **Split Strategy:** Partial classes (`.Core.cs`, `.Mouse.cs`, `.Keyboard.cs`, `.ImagePixel.cs`).

### ScriptViewModel → 3 classes:

**ScriptViewModel.cs** (600 lines) — Core execution
- Event list management
- Execution orchestration
- Script loading

**RecordingService.cs** (300 lines) — Recording only
- Start/stop recording
- Event capture
- Screen capture triggers

**ScriptExecutionManager.cs** (400 lines) — Execution details
- Step execution
- Delay handling
- Error recovery

### ScriptEditorViewModel.cs → 4 classes:

**ScriptEditorViewModel.cs** (800 lines) — Core editing
- Code text management
- Tab switching
- Basic editor commands

**EditorPreviewManager.cs** (400 lines) — Previews only
- Preview generation
- Preview refresh
- Preview state

**UndoRedoManager.cs** (300 lines) — Undo/redo
- Undo stack
- Redo stack
- State snapshots

**AiCodeGenerator.cs** (500 lines) — AI integration
- GPT/OpenAI calls
- Code generation
- Prompt management

## Splitting Strategy

1. Extract smallest responsibility first (RecordingService)
2. Update imports in all files
3. Build and test
4. Repeat for next extraction
5. Do NOT split all at once — one extraction per build cycle

## Expected Impact

- Each class < 600 lines
- Single responsibility per class
- Easier to test individual features
- Safer async void management

## Risk Mitigation

- One extraction at a time
- Build after each extraction
- No behavior changes — just moving code
- Can revert individual extractions

---

**Awaiting Review:** Other agent to confirm "Agree" or provide counterargument.
