---
purpose: Common Agent Mistakes & Project-Specific Traps
project: PowerX Keys
version: 1.3
date: 2026-07-09
instructions: >
  READ THIS early in your session (after SOUL.md + MEMORY.md).
  APPEND to this when you discover a new gotcha.
  This file prevents agents from making the same mistake twice.
---

# ⚠️ GOTCHAS.md — Common Traps & Agent Mistakes

> **Why this exists:** Agents keep making the same mistakes because they're stateless. This file is the permanent record of codebase-specific traps.
>
> **When to read:** Every session, after SOUL.md and MEMORY.md.

---

## 🔥 Critical Gotchas (Will Break Things)

### G-001: AHK v1 vs v2 Syntax
**Trap:** 90% of AutoHotkey content online is for AHK **v1**. PowerX Keys uses AHK **v2**, which has fundamentally different syntax.
**What goes wrong:** Generating v1 syntax (e.g., `MsgBox, text` instead of `MsgBox "text"`, `%var%` instead of `var`) causes silent script failures.
**Fix:** Always include "AutoHotkey v2" or "AHK v2" in web searches. See `ahk-scripting.skill.md` for the v1→v2 conversion table.

### G-002: PowerX_Engine.exe Is Just AHK
**Trap:** Agents sometimes assume `PowerX_Engine.exe` is a custom engine binary.
**What goes wrong:** Agents may try to "debug" it, look for source code, or treat it as a separate application.
**Reality:** It is literally the standard `AutoHotkey64.exe` renamed for a cleaner Task Manager appearance. It has no custom C# code.

### G-003: MacroEditorViewModel Is 6 Files, Not 1
**Trap:** Searching for `MacroEditorViewModel.cs` and opening only the first result.
**What goes wrong:** You edit the wrong partial class file. Properties are in `Properties.cs`, commands in `Commands.cs`, recording in `Recording.cs`, etc.
**Fix:** Check `wpf-patterns.skill.md` → "MacroEditorViewModel — Partial Class Map" before editing.

### G-004: Mouse Trace Bugs Are in C#, Not AHK
**Trap:** Looking in AHK scripts for mouse trace / SmoothTrace bugs.
**Reality:** Mouse trace playback uses C# `SetCursorPos` via P/Invoke (`SmoothTraceEngine.cs`). AHK only handles hotkeys, image search, and pixel search.
**Fix:** Mouse trace issue → `SmoothTraceEngine.cs`. Hotkey issue → `ScriptCompilerService.cs`.

### G-005: WPF Popup Visual Tree Is Detached
**Trap:** Using `VisualTreeHelper.GetParent()` inside a Popup to find the parent `ActionItem`.
**What goes wrong:** Popups have a detached visual tree — walking up from inside a Popup never reaches the `DataContext` of the item that opened it.
**Fix:** When inside a Popup, use `popup.PlacementTarget` to get the gear button, then walk up from there to find the `ActionItem` in `DataContext`. See `FindActionItemAncestor()` in `ScriptLibraryView.xaml.cs`.

### G-006: WIN_SMART Scope Not Handled in Main Compiler
**Trap:** Adding a new scope type (like `WIN_SMART:`) to the preview compiler (`ScriptCompilerService.SingleStep.cs`) but forgetting the main macro compiler (`ScriptCompilerService.cs`).
**What goes wrong:** Preview works perfectly (Smart Box rebases correctly), but when the macro actually runs, it falls through to static coordinates and only Full Screen fallback finds the image. Extremely confusing to debug because "preview works but execution doesn't."
**Fix:** Both compilers must handle the same scope strings. Check line ~2068 in `ScriptCompilerService.cs` and line ~77 in `SingleStep.cs` — keep them in sync. See HANDOFF.md "Two Compilers" section.

### G-007: Multi-Window Source Detection Picks Wrong Instance
**Trap:** Using `Process.GetProcessesByName()` and grabbing the first match to detect which window was captured.
**What goes wrong:** If the user has multiple windows of the same app open (e.g. two Chrome windows, two instances of any app), the code grabs the position of the wrong one. Smart Box then gets placed at incorrect coordinates, fails, and falls back to Window search.
**Fix:** Use the window handle from `WindowFromPoint` (already captured during overlay interaction) to get the exact window rect. Don't loop through processes after the fact — the overlay already knows the right handle.


### G-008: AHK v2 PixelSearch Expects RGB (Not BGR) Color Format
**Trap:** Converting WPF `#RRGGBB` hex strings into `0xBBGGRR` (BGR) format before calling AutoHotkey v2 `PixelSearch`.
**What goes wrong:** AHK v2 `PixelSearch` defaults to **RGB (`0xRRGGBB`)** format. Passing BGR causes AHK to search for Blue on Red pixels (and vice versa), failing 100% of the time on single colored pixels or thin line strokes.
**Fix:** Pass exact RGB hex (`0xRRGGBB`) to `PixelSearch` in `ScriptCompilerService.cs` and `ScriptCompilerService.SingleStep.cs`.

### G-009: Scope GUI Overlay Obscuring Screen Pixels Before Search
**Trap:** Showing an `+AlwaysOnTop` scope boundary GUI window (`_scopeGui`) before calling `PixelSearch` in single-step preview.
**What goes wrong:** The cyan/solid scope overlay window physically paints over the target pixel on screen before `PixelSearch` samples the display, causing `PixelSearch` to read cyan color (`#00CED1`) instead of the true screen color.
**Fix:** Execute `PixelSearch` *before* showing any scope overlay GUIs in preview scripts.

---

```markdown
### G-NNN: [Short descriptive title]
**Trap:** What the agent might do wrong
**What goes wrong:** The consequence
**Fix:** The approach
```

Increment the number (G-005, G-006, etc.) and place it in the appropriate severity section.

---

## Related Files
- [[SOUL]] — Agent identity and hard boundaries
- [skills/code-review.skill.md](file:///C:/Users/Maaz/Documents/New%20folder/Obsidian%20Vault/skills/code-review.skill.md) — Review checklist catches some of these
