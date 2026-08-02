---
type: decision
status: active
summary: Architecture Decision Record for white-labeled AutoHotkey v2 execution engine.
last_updated: 2026-07-28
---

# 📜 ADR 0001: AutoHotkey v2 White-Labeled Engine Architecture

## Context
PowerX Keys requires high-speed, reliable low-level Windows input automation (mouse clicks, keystrokes, image search, pixel detection, window activation).

## Decision
1. Macro configurations are stored as JSON in C#.
2. `ScriptCompilerService.cs` compiles JSON macro structures into clean AutoHotkey v2 (`.ahk`) scripts.
3. The generated script is passed to `PowerX_Engine.exe` (a white-labeled AHK v2 runner).
4. **Golden Rule**: Never edit `.ahk` scripts directly. Always modify `ScriptCompilerService.cs`.

## Rationale
* Native AHK v2 handles Windows hooks, global hotkeys, and GDI pixel searches with sub-millisecond latency.
* White-labeling `PowerX_Engine.exe` hides the raw script execution from end users while maintaining standalone portable capability.
