# 🔍 Deep Audit Prompt 4: All Action & Logic Blocks Comprehensive Audit

You are tasked with executing a deep structural and execution audit of all Macro Step Action and Logic Blocks in **PowerX Keys**.

---

## 💡 Research Strategy
- **Search Obsidian Vault First**: Search `c:\Users\Maaz\Documents\New folder\Obsidian Vault\` for existing documentation, GOTCHAS, and schemas before doing full codebase searches.
- **Use Graphify Knowledge Graph**: Check `graphify-out/` or use `graphify query` for fast node/relationship lookup across components.

---

## 🎯 Target Scope
- **Root Directory**: `c:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\`
- **Core Files**: `PowerX.Core\Models\MacroStep.cs`, `PowerX.Services\Services\ScriptCompilerService.cs`, `PowerX.Services\Services\ScriptCompilerService.SingleStep.cs`, `PowerX.Services\Services\MacroExecutionService.cs`

---

## 📋 Comprehensive Audit Checklist

### 1. Complex Search & Targeting Blocks
- **Window Action**: Active window vs specific window title, window actions (Restore/Minimize/Close/Focus), coordinate pulsing.
- **Image Search**: Image capture, Turbo Mode (FastEngine), tolerance sliders, smart search area, preview sonar.
- **Pixel Search**: Color picking, tolerance variance, target offset, preview sonar.
- **UI Element**: Hover highlight capture, `Ctrl` lock, `UIAutomationId`, `ControlType`, fallback coordinates.

### 2. Standard Action & System Blocks
- **Mouse Click**: Single/double/right click, hold/release, target offset, playback speed override.
- **Keyboard / Text**: Key combo capture, humanized typing delay, special character brace escaping.
- **Delay / Wait**: Fixed delay, randomized delay, Wait Until, Wait for Key.
- **File Launcher / Sound / Popup / Notification**: Audio alert WAV playback, modal check popups, system tray notifications.

### 3. Logic & Container Blocks
- **Logic IF**: True/False branch execution, swap branches command, nested logic blocks, `LastActionSucceeded` evaluation.
- **Loop / Group**: Loop count, Infinite loop, break condition, child step nesting and unnesting.

---

## 📝 Output Requirement
Please write your detailed findings, verified root causes, line numbers, and actionable bug reports directly to:
`c:\Users\Maaz\Documents\New folder\audit_report_4.md`
