---
tags: [audit, qa, blocks]
date: 2026-07-05
author: AI Agent
status: completed
---

# 🔍 Keyboard Block — QA Findings

**Summary:** Full audit of the Keyboard block. Everything works except one cosmetic issue.

## ✅ All Working

- **Value** (key assignment) — model, UI, compiler, clone all good
- **KeyActionType** (Press / Hold Down / Released Up) — tri-state cycles correctly, compiler generates correct AHK for all 3 states in both normal & sandbox mode
- **IsValid** — warning dot shows when key is empty or "None"
- **DisplayValue** — shows "Press/Hold/Release + key name" correctly
- **Combo keys** (e.g. Ctrl+C) — handled via `ConvertComboToAhk` and `ConvertComboHoldReleaseToAhk`
- **Clone/Duplicate** — all keyboard-specific props preserved
- **StepName** — defaults to "Keyboard" on creation

## ⚠️ Issue Found

### "Mimic Human Timing" checkbox does nothing on Keyboard blocks

- **What:** The `IsHumanized` checkbox appears in the Keyboard block UI (KeyboardInputTemplates.xaml, line 462)
- **But:** The compiler (`ScriptCompilerService.cs`) only uses `IsHumanized` for **Delay** blocks (line 1682) and **MouseClick** drag blocks (line 1799) — never for `MacroStepType.Keyboard`
- **Result:** User can toggle it on, it saves/loads/clones fine, but it has **zero effect** on the generated AHK script
- **Fix options:**
  1. Add humanized delay before/after the `Send()` call in the keyboard compiler section (~line 1692)
  2. Or hide the checkbox from the keyboard block UI if humanization isn't intended for keypresses

---

### 🐛 Sandbox: Combo + Hold/Release silently ignored

- **What:** In sandbox mode (`ScriptCompilerService.cs`, ~line 4563), combo keys (e.g. `Ctrl + C`) with Hold Down or Released Up toggle **always send a normal Press instead**
- **Why:** Sandbox calls `ConvertComboToAhk` (line 4568) but **never calls `ConvertComboHoldReleaseToAhk`**. Since `combo` is not null for combos, line 4598 always sends `combo` (which is the Press version like `^c`)
- **Normal compile is fine** — it correctly calls `ConvertComboHoldReleaseToAhk` at line 1697 and handles all 3 states
- **Affected:** Only sandbox/test mode. Production compile works correctly
- **Fix:** Add `ConvertComboHoldReleaseToAhk` call in the sandbox keyboard section, mirroring the normal compile logic at lines 1696–1706

---

## Type Text Block

### ✅ All Working

- **Value** (typed text) — model, UI popup, compiler all good
- **IsFastPasteMode** (Write vs Paste toggle) — both modes compile correctly to AHK (clipboard paste vs SendEvent)
- **UseVariable** + **VariableSource** — compiler handles variable-linked text properly
- **IsValid** — checks Value or VariableSource depending on UseVariable toggle
- **DisplayValue** — shows "Type: text" or "Type Var: {name}" correctly
- **Clone/Duplicate** — all text-specific props preserved
- **StepName** — defaults to "Type Text" on creation
- **IsHumanized** — works in C# execution path (MacroExecutionService line 1381, per-character delay), but not in AHK compiled path

### ⚠️ Issues Found

#### `InitialCapsLock` — dead property

- **What:** Property exists in model (line 528), clones (line 1726), serializes — but **compiler never reads it** and there's no UI for it on the Text block
- **Fix:** Either wire it into the compiler (toggle CapsLock state before typing) or remove the property

#### Sandbox doesn't handle Text blocks

- **What:** Sandbox compilation only supports Delay, MouseClick, Keyboard, GroupHeader, CallMacro — Text block is **completely skipped**
- **Fix:** Add Text block handling in sandbox (~line 4600 in ScriptCompilerService.cs)

---

## Wait Block (WaitForKey + WaitUntil)

### ✅ WaitForKey — All Working

- **WaitContinueKey** (default: Enter) — model (line 674), UI bound (line 887, 1005), compiler uses it (line 2871), clone (line 1713) ✅
- **WaitCancelKey** (default: Escape) — model (line 686), UI bound (line 913, 1029), compiler uses it (line 2872), clone (line 1714) ✅
- **WaitKeyMode** (3 modes: Specific / StrictOK / StrictCancel) — model (line 699), ComboBox bound (line 950, 1078), compiler handles all 3 modes (lines 2956–3038), clone (line 1715) ✅
- **WaitMessageType** (ToolTip vs Popup) — model (line 706), ComboBox bound (line 981, 1096), compiler generates both ToolTip GUI and Popup GUI (lines 2878–3043), clone (line 1716) ✅
- **ShowWaitMessage** — model (line 769), checkbox bound (line 955, 1083), compiler conditionally adds message display, clone (line 1717) ✅
- **Value** (display message text) — auto-updates when keys change via `AutoUpdateWaitForKeyMessage` (line 712) ✅
- **DisplayValue** — shows "Enter / Escape" format correctly (line 389) ✅
- **IsValid** — requires non-empty Value (line 1114) ✅
- **Initialization** — proper defaults set: Enter/Escape keys, ToolTip mode, ShowWaitMessage=false (lines 163–169) ✅
- **Token cancellation** — all 3 modes + popup properly check `myToken != GlobalMacroToken` for stop support ✅

### ✅ WaitUntil — All Working

- **WaitConditionType** (ImageFound / PixelFound / WindowActive) — model (line 1335), compiler handles all 3 (line 3880), clone (line 1757) ✅
- **Duration** (timeout) — used as timeout in ms (line 3881) ✅
- **CheckIntervalMs** — model (line 1342), compiler uses it (line 3882), clone (line 1758) ✅
- **OnTimeoutAction** (Stop / Continue) — model (line 1349), compiler (line 3883), clone (line 1759) ✅
- **DisplayValue** — shows condition-specific text (lines 336–339) ✅
- **IsValid** — validates per condition type: image needs filename, pixel needs color+coords, window needs value (lines 1117–1120) ✅
- **Initialization** — proper defaults: ImageFound, 5000ms timeout, 250ms interval, Stop on timeout (lines 181–190) ✅

### ✅ No Issues Found

- Both WaitForKey and WaitUntil are **not in sandbox** — but that's expected since they're interactive/conditional blocks that don't make sense to sandbox-test
- All properties fully wired: model → UI → compiler → clone ✅

---

## Cross-Block Interactions

### ⚠️ Keyboard Hold + Text Paste = broken Ctrl+V

- **Scenario:** Keyboard block (Shift, Hold Down) → Text block (Paste mode)
- **What happens:** Shift Hold compiles to `Send("{Blind}" . "{Shift Down}")`. Then Paste mode compiles to `Send("^v")`. Since Shift is still held at the OS level, the actual keypress becomes **Shift+Ctrl+V** instead of Ctrl+V
- **Impact:** Some apps treat Shift+Ctrl+V differently (e.g., Chrome = "Paste without formatting"). Others may ignore it entirely
- **Write mode is fine:** `SendEvent("{Text}" . "...")` uses Unicode character input which bypasses modifier states — text types as-is ✅
- **Fix:** In the Text block compiler, temporarily release held modifiers before `Send("^v")` and restore after, or use `Send("{Blind}^v")`

---

## Mouse Block

### ⚠️ Issues Found

#### `CanReleaseMouse` — dead property

- **What:** Property exists in model (line 494), clones (line 1696), serializes — but **compiler never reads it**
- **Fix:** Either wire it in or remove it

#### `IsHumanized` only works for drag, not regular clicks

- **What:** Checkbox appears in UI for all mouse blocks (line 591, 776), but compiler only uses it for **Drag and Drop** (line 1799). Regular clicks, scrolls, and moves **ignore it**
- **Fix:** Add humanized delay before Click() for non-drag actions too

#### Sandbox ignores all mouse properties except X/Y

- **What:** Sandbox (lines 4556–4562) does a bare `Click(x, y)` or `Click()` — ignores button type, scroll amount, click count, action target, coordinate mode, KeyActionType, everything
- **Fix:** Mirror at least the button type and basic action handling from the normal compiler

---

## Popup (Message) Block

### ⚠️ Issue Found

#### `PopupMode` completely ignored by compiler

- **What:** UI lets users select "Checkpoint" (OK/Cancel), "Auto-Timeout" (dismiss after X seconds), and "Floating Alert" (non-blocking). But the **compiler always generates a basic `MsgBox("msg", "title", "OC")`** regardless of mode (line 2808)
- **Impact:** "Auto-Timeout" and "Floating Alert" modes **do nothing different** — they all show the same OK/Cancel dialog
- **Fix:** Implement different AHK output for each mode: Auto-Timeout → `MsgBox` with timer, Floating Alert → `ToolTip` or non-blocking GUI

---

## File Launcher (Open/Run) Block

### ⚠️ Issue Found

#### FileLauncher compiler missing

- **What:** `MacroStepType.FileLauncher` is **completely missing** from `ScriptCompilerService.cs`. The compiler has no case for it — it's silently skipped
- **Impact:** Users can add the block, configure a file path, but the block **does nothing** when the macro runs
- **Fix:** Add `Run("file_path")` compilation in the compiler, and `Process.Start()` in execution service

---

## Set Variable Block

### ⚠️ Issue Found

#### No IsValid validation

- **What:** `MacroStepType.SetVariable` is **not checked** in the `IsValid` property. Falls through to default `return true`, meaning a SetVariable block with empty variable name or empty value is considered "valid"
- **Impact:** No red warning dot when the variable name is blank
- **Fix:** Add `if (Type == MacroStepType.SetVariable) return !string.IsNullOrWhiteSpace(InputVariableName);`

---

## Image Search Block

### 🐛 Issues Found

#### `WIN_SMART:` scope not handled in compiler

- **What:** Line 1927 only handles `WIN_LIVE:` and `WIN_REL:`. The `WIN_SMART:` scope falls through to the generic `else` branch (line 2006) which strips "WIN:" and splits by comma — this produces garbage coordinates
- **Impact:** Users who set a WIN_SMART scope on an ImageSearch step get broken search areas. **PixelSearch correctly handles WIN_SMART (line 2210)**
- **Fix:** Add `|| step.SearchScopeSummary.StartsWith("WIN_SMART:")` to the condition at line 1927, with full-window scope + fallback logic

#### `AutoLaunchPath` missing from Clone (shared)

- **What:** `AutoLaunchPath` (model line 588) is NOT included in the Clone method. Duplicating an ImageSearch step with a custom auto-launch path silently drops the path
- **Fix:** Add `AutoLaunchPath = this.AutoLaunchPath,` to Clone method

---

## Pixel Search Block

### ⚠️ Shared Issue Only

#### `AutoLaunchPath` missing from Clone (shared with all vision blocks)

---

## Window Action Block

### ⚠️ Shared Issue Only

#### `AutoLaunchPath` missing from Clone (shared with all vision blocks)

---

## UI Element Block

### 🐛 Issues Found

#### SameApp/UIMatchByProcess not compiled for AHK scripts

- **What:** Main compiler (line 3067) always uses `WinExist("windowTitle")` — never checks `UIMatchByProcess` or `UIFindMode == "SameApp"` to use `UIProcessName` with `ahk_exe`
- **Works in:** Sandbox preview (line 584) and execution service (ResolveWindowHandle) — but NOT in compiled AHK
- **Impact:** SameApp mode works in preview but fails in compiled scripts if window title changes
- **Fix:** Add SameApp check from SingleStep.cs before the WinExist() call in the main compiler

---

## Rescan Findings — Round 2

### 🐛 NEW: Keyboard `Ctrl+{` or `Ctrl+}` produces broken AHK

- **What:** `ConvertComboToAhk()` line 132: `key.Length == 1 ? key : "{" + key + "}"`. If final key is literally `{` or `}`, it produces `Send("^{")` — unmatched braces = AHK syntax error
- **Fix:** Escape as `{{}` and `{}}` for brace characters

### 🐛 NEW: Mouse "Release Up" not recognized

- **What:** Mouse block (line 1768) only checks `"Released Up"` but Keyboard block (line 1716) checks BOTH `"Released Up" || "Release Up"`. If UI sets `"Release Up"` for mouse, the Hold/Release logic won't trigger
- **Fix:** Add `|| keyAction == "Release Up"` to the mouse block check

### 🐛 NEW: Popup message/title escaping incomplete

- **What:** Lines 2806–2807 only escape double quotes. Backticks, newlines, tabs are NOT escaped. A message containing `` ` `` or `\n` would break the AHK MsgBox call
- **Fix:** Use `EscapeStringForAhkLiteral()` instead of manual `.Replace()`

---

## Cross-Block: ImageSearch + LogicIf + Click Flow

### ⚠️ Edge Case Found

#### Stale FoundX/FoundY on failure — no clearing

- **What:** When ImageSearch **fails**, `FoundX/FoundY` are NOT cleared — they keep their last successful values (initialized to `0,0` at macro start). So if ImageSearch 1 succeeds, then ImageSearch 2 fails, `FoundX/FoundY` still point to Image 1's location
- **Impact:** The click targets the wrong location silently. The execution service (`_lastFoundX/_lastFoundY`) DOES clear on failure (line 1544), but the AHK compiler does NOT
- **Fix:** Add `FoundX := 0, FoundY := 0` before each ImageSearch/PixelSearch block in the compiler, matching the execution service behavior

---

## Cross-Block: 9 Real-World Patterns Tested

### ✅ Verified Clean — No Issues

| Pattern | Result |
|---|---|
| WindowAction Activate → ImageSearch → Click | ✅ Activation completes before search runs, scopes are independent |
| UserInput → SetVariable → LogicIf VariableEquals | ✅ All 3 blocks use identical sanitization, same UserVariables map |
| Multiple Named ImageSearches → Click at Specific | ✅ Named prefixes properly isolated, sanitization consistent |
| Loop → ImageSearch → LogicIf → Different actions | ✅ LastActionSucceeded resets each iteration, StepSuccessStates overwrite correctly |
| Notification + Sound in fast Loop | ✅ TrayTip overwrites (no stacking), SoundBeep is synchronous |
| UIElement ReadText → SetVariable → LogicIf | ✅ All use UserVariables map. User must match names manually (StepName vs InputVariableName) |
| PixelSearch → LogicIf → ImageSearch in else | ✅ Both set FoundX/FoundY, both reset LastActionSucceeded, else branch works |

---

## Preview / Execution Service Deep Scan

All 23 blocks checked for preview vs compiler parity in `MacroExecutionService.cs`.

### 🐛 Preview Bugs Found

#### HIGH: UserInput uses StepName instead of InputVariableName

- **What:** Compiler (line 2816) stores input into `UserVariables["InputVariableName"]`. Preview (line 1456) stores into `_readTextValues["StepName"]`. These are **different properties**
- **Impact:** If user sets InputVariableName = "MyVar" and StepName = "Step1" — compiled mode stores into `MyVar`, preview stores into `Step1`. Any downstream block reading `MyVar` works in compiled mode but fails in preview
- **Fix:** Change preview to use `step.InputVariableName` instead of `step.StepName`

#### HIGH: WaitUntil completely missing from preview

- **What:** No `case MacroStepType.WaitUntil` exists in `MacroExecutionService.cs`. Falls through to `default: return true` — instantly succeeds without waiting for ANY condition
- **Impact:** Macros with WaitUntil (ImageFound, PixelFound, WindowExists, WindowActive) behave completely differently in preview vs compiled — preview skips the wait entirely
- **Fix:** Add WaitUntil case with polling loop for all condition types

#### MEDIUM: Mouse "Active Window" ActionTarget missing from preview

- **What:** Compiler (line 1824) generates `WinGetPos` and clicks center of active window. Preview has no `"Active Window"` check — falls through to saved X/Y coordinates or current mouse position
- **Impact:** Click at "Active Window" center works compiled but clicks wrong location in preview
- **Fix:** Add `else if (step.ActionTarget == "Active Window")` with `GetForegroundWindow()` + `GetWindowRect()` center calculation

---

## 🔎 Cross-Agent Verification (Agent 86050fc7 — 2026-07-07)

Independently verified 10 bugs from this document. **8 confirmed, 2 are wrong.**

| # | Bug Claim | Verdict | Evidence |
|---|-----------|---------|----------|
| 1 | FileLauncher not compiled | ❌ **WRONG** — It IS compiled | ScriptCompilerService.cs lines 2869-2880 generates `Run()` properly |
| 2 | PopupMode ignored by compiler | ✅ Confirmed | Line 2808 always uses `"OC"` regardless of mode |
| 3 | Mouse Multi-Click always Left Click | ✅ Confirmed | Lines 1810-1816 exclude Multi/Timer from `clkParam` |
| 4 | SetVariable no IsValid | ✅ Confirmed | Falls through to `return true` at line 1127 |
| 5 | WaitUntil doesn't set FoundX/FoundY | ❌ **WRONG** — It DOES set them | Lines 4075-4076 copy `_WaitX/_WaitY` to `FoundX/FoundY` |
| 6 | UserInput preview uses wrong var | ✅ Confirmed | Line 1468 uses `StepName` instead of `InputVariableName` |
| 7 | WaitUntil missing from preview | ✅ Confirmed | No case in ExecuteStepAsync, falls to default true |
| 8 | AutoLaunchPath missing from Clone | ✅ Confirmed | Not in Clone method despite line 1741 copying the bool flag |
| 9 | Sandbox ignores LoopSequence | ✅ Confirmed | No case in CompileSandboxStep (lines 4565-4644) |
| 10 | FailIfMissing inside Loop kills macro | ✅ Confirmed | Line 2162 uses `return` which exits entire function |

### Corrections Needed in This Document

> [!WARNING]
> **Bug #1 (FileLauncher):** Remove the "CRITICAL: Not compiled at all!" label (line 260). The compiler DOES handle it at lines 2869-2880. The block works in production.

> [!WARNING]
> **Bug #5 (WaitUntil FoundX/FoundY):** Remove this claim (line 655-660). The compiler correctly copies `_WaitX/_WaitY → FoundX/FoundY` at lines 4075-4076. A Mouse Click after WaitUntil WILL click the right spot.

### Awaiting Opinion From Original Author (Agent ed424685)