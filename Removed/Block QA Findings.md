---
tags: [feature, qa, keyboard]
date: 2026-07-05
status: pending-fix
---

# Keyboard Block — QA Findings

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

### ✅ Keyboard Hold + Mouse Click = intentional (no bug)

- Shift/Ctrl/Alt Hold → Mouse Click is a **valid use case** (Shift+Click for range select, Ctrl+Click for multi-select, etc.)
- Applications receive the modifier state with the click — this is expected behavior

### ⚠️ Keyboard Hold + Mouse Scroll = silent behavior change

- **Scenario:** Keyboard Shift Hold → Mouse Scroll Up/Down
- **What happens:** Many apps treat **Shift+Scroll = horizontal scroll** instead of vertical
- **Not a bug** in the compiler, but users may not realize their scroll direction changes when a Shift Hold precedes it
- **Suggestion:** Consider a UI tooltip warning when Shift Hold + Scroll are adjacent

### ⚠️ Mouse Hold Down without matching Release = ghost mouse button

- **Scenario:** User sets Mouse block to "Left Click" with KeyActionType = "Hold Down" but forgets to add a matching "Released Up" block
- **What happens:** Mouse button stays held at OS level. The macro-end safety (line 1569) only releases **keyboard modifiers** (`Shift Up`, `Ctrl Up`, `Alt Up`, `LWin Up`, `RWin Up`) — it does **NOT release mouse buttons**
- **Fix:** Add `Click("Up")` to the macro cleanup at line 1569

### ✅ Keyboard Hold without Release = handled by ghost reset

- **Scenario:** User sets Keyboard Shift Hold but forgets a matching Release
- **What happens:** The "Ghost Keyboard Reset" at line 355–360 runs `OnExit` and releases all modifiers. Also, macro completion (line 1569) releases all modifiers. **Covered** ✅

### ✅ CoordinateMode leak between Mouse blocks = handled

- **Scenario:** Mouse block 1 uses Window mode, Mouse block 2 uses Screen mode
- **What happens:** Compiler resets CoordMode back to "Screen" after each non-Screen mouse block (line 1892–1894). No leak ✅

### ✅ Text Paste clipboard restoration

- **Scenario:** Two Text Paste blocks in sequence
- **What happens:** Each block independently saves clipboard via `ClipboardAll()`, pastes, then restores. Sequential execution means no conflict ✅
- **Edge case:** 150ms restore delay (line 2543) is tight for slow apps but acceptable

### ⚠️ CallMacro inherits parent's modifier state (no isolation)

- **Scenario:** Macro A has Keyboard Ctrl Hold → Call Macro B → Macro B has a Mouse Click
- **What happens:** CallMacro uses **inline expansion** (line 4115: `CompileStepCollection`), not a function call. So Macro B's steps execute with Macro A's Ctrl still held — the click becomes Ctrl+Click
- **This is by design** (inlining is intentional), but users likely don't realize their sub-macro inherits held modifiers
- **Suggestion:** Consider a compiler warning when CallMacro follows a Hold block without a matching Release

### ✅ Loop + Keyboard Hold = harmless (no stacking)

- **Scenario:** Loop 5x containing Keyboard Shift Hold → Type Text
- **What happens:** Each iteration sends `{Shift Down}` again, but AHK treats repeated `{Key Down}` as a no-op if already held. Text types fine each iteration. No stacking bug ✅

---

## Mouse Block

### ✅ All Working

- **Value** (click type: Left Click, Right Click, Double Click, Scroll Up/Down, Drag and Drop, Move Only, Multiple Clicks, Timer Click, Mouse 4/5) — model, UI dropdown, compiler handles all types correctly ✅
- **X/Y coordinates** — model (line 189–198), compiler uses them for Click and MouseMove (line 1830), clone (line 1688–1689) ✅
- **EndX/EndY** (drag target) — compiler uses for drag-and-drop (lines 1793–1796), validation requires them for drag types (line 1068) ✅
- **ActionTarget** (Coordinates / Found Image / Pixel Color / Specific Image/Pixel / Active Window / No Target) — UI dropdown (line 489), compiler handles all targets (lines 1823–1830) ✅
- **CoordinateMode** (Screen / Window) — compiler sets `CoordMode` and resets after (lines 1741–1895) ✅
- **CoordinateWindowTitle** — compiler activates target window before click (lines 1744–1750) ✅
- **KeyActionType** (Press / Hold Down / Released Up) — works for mouse too! Adds " Down"/" Up" to button arg (lines 1765–1768), correctly skips for scrolls and drags ✅
- **ScrollAmount** — model (line 303), UI bound (line 623, 702), compiler uses it (line 1758–1759), DisplayValue shows "Scroll Up (x3)" format ✅
- **ClickCount** — model (line 918), UI bound (line 466, 947), compiler uses for Multiple Clicks and Timer Click (line 1858) ✅
- **TimerInterval / DoubleClickSpeed** — compiler uses correct one per action type (line 1859), respects PlaybackSpeed ✅
- **IsMouseVisibleMove** — compiler uses for animated mouse movement (lines 1835, 1847) ✅
- **IsMouseReturnToOrigin** — compiler saves/restores mouse position (lines 1838–1889) ✅
- **IsHumanized** — works for **drag-and-drop** (line 1799), adds humanized delay before drag ✅
- **DisplayValue** — friendly names: "Left Click" → "Click", scroll shows amount (lines 433–443) ✅
- **IsValid** — comprehensive: validates per action type, checks coords for drags, Timer Click needs count+interval, Multiple Clicks needs count (lines 1040–1070) ✅
- **Initialization** — defaults to "Left Click" + "Coordinates" target (lines 107–108) ✅
- **Clone** — all mouse props preserved (lines 1688–1730) ✅

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

### ✅ All Working

- **Value** (message text) — model, UI textbox, compiler uses it (line 2806) ✅
- **WindowTitle** — model, compiler uses it as MsgBox title (line 2807) ✅
- **DisplayValue** — shows `PopupMode` (e.g. "Checkpoint") correctly (line 377) ✅
- **IsValid** — requires non-empty Value (line 1092) ✅
- **StepName** — defaults to "Message" on creation (line 235) ✅
- **Clone** — PopupMode, PopupTimeout both preserved (lines 1734–1735) ✅

### ⚠️ Issue Found

#### `PopupMode` completely ignored by compiler

- **What:** UI lets users select "Checkpoint" (OK/Cancel), "Auto-Timeout" (dismiss after X seconds), and "Floating Alert" (non-blocking). But the **compiler always generates a basic `MsgBox("msg", "title", "OC")`** regardless of mode (line 2808)
- **Impact:** "Auto-Timeout" and "Floating Alert" modes **do nothing different** — they all show the same OK/Cancel dialog
- **Fix:** Implement different AHK output for each mode: Auto-Timeout → `MsgBox` with timer, Floating Alert → `ToolTip` or non-blocking GUI

---

## Notification Block

### ✅ All Working

- **Value** (notification text) — model, UI, compiler uses it (line 3721) ✅
- **WindowTitle** — compiler uses it as TrayTip title (line 3722) ✅
- **NotificationIcon** (Info / Warning / Error) — compiler maps to TrayTip options flags (lines 3724–3727) ✅
- **NotificationSilent** — adds flag 16 for silent mode (line 3729) ✅
- **PopupTimeout** — auto-dismisses TrayTip via `SetTimer` (lines 3759–3761) ✅
- **SoundType + SoundFilePath** — notification can play a sound alongside the tray tip (lines 3734–3756) ✅
- **DisplayValue** — shows notification text (line 381) ✅
- **IsValid** — requires non-empty Value (line 1093) ✅
- **StepName** — defaults to "Notify" (line 211) ✅
- **Clone** — NotificationIcon, NotificationSilent preserved (lines 1736–1737) ✅

### ✅ No Issues Found

---

## Play Sound Block

### ✅ All Working

- **SoundType** — model (line 945), UI dropdown, compiler handles all types: Custom Beep (Short/Medium/Long), Double Alert, Success Chime, Error Chord, Default Sound, Custom File (lines 3776–3799) ✅
- **SoundFilePath** — compiler checks `FileExist()` before playing, with 10s auto-stop timer (lines 3766–3772) ✅
- **DisplayValue** — shows SoundType name (line 399) ✅
- **IsValid** — requires SoundType, Custom File also requires SoundFilePath (lines 1094–1098) ✅
- **StepName** — defaults to "Play Sound" (line 215) ✅
- **Clone** — SoundType, SoundFilePath preserved (lines 1731–1733) ✅
- **Transfer/Export** — custom sound files are included in macro exports (MacroTransferManager line 399) ✅

### ✅ No Issues Found

---

## File Launcher (Open/Run) Block

### 🐛 CRITICAL: Not compiled at all!

- **Value** (file path) — model, UI textbox + browse button (line 1111–1131 in KeyboardInputTemplates.xaml) ✅
- **DisplayValue** — shows filename via `Path.GetFileName(Value)` (line 394) ✅
- **IsValid** — requires non-empty Value (line 1111) ✅
- **StepName** — defaults to "Open / Run" (line 227) ✅
- **Clone** — Value is a base property, always preserved ✅

### ⚠️ Issues Found

#### FileLauncher not in compiler — block does NOTHING in production

- **What:** `MacroStepType.FileLauncher` is **completely missing** from `ScriptCompilerService.cs`. The compiler has no case for it — it's silently skipped
- **Also missing from:** `MacroExecutionService.cs` — no execution path either
- **Impact:** Users can add the block, configure a file path, but the block **does nothing** when the macro runs
- **Fix:** Add `Run("file_path")` compilation in the compiler, and `Process.Start()` in execution service

---

## User Input Block

### ✅ All Working

- **Value** (prompt text) — model, UI, compiler uses it as InputBox/GUI prompt (lines 2814–2859) ✅
- **WindowTitle** — compiler uses it as dialog title (line 2815) ✅
- **InputVariableName** — compiler sanitizes and stores in `UserVariables[varName]` (line 2816) ✅
- **InputType** (Text / YesNo / Dropdown) — compiler generates different AHK GUIs for each (lines 2818–2859):
  - Text → `InputBox()` ✅
  - YesNo → custom GUI with two buttons ✅
  - Dropdown → custom GUI with `DropDownList` ✅
- **InputOptions** — used for YesNo button labels and Dropdown items (lines 2821–2852) ✅
- **Cancel handling** — Text mode returns on Cancel (line 2858), YesNo/Dropdown use `WinWaitClose` ✅
- **DisplayValue** — shows "Text input" / "YesNo input" etc. (line 385) ✅
- **IsValid** — requires non-empty Value (line 1108) ✅
- **StepName** — defaults to "User Input" (line 219) ✅
- **Clone** — InputVariableName, InputType, InputOptions all preserved (lines 1706–1708) ✅
- **Execution service** — preview mode shows real WPF dialogs (lines 1402–1464) ✅

### ✅ No Issues Found

---

## Set Variable Block

### ✅ Mostly Working

- **InputVariableName** — model (line 610), compiler sanitizes and stores in `UserVariables[varName]` (line 2864) ✅
- **Value** — compiler uses as the variable value (line 2866) ✅
- **DisplayValue** — shows `"varName = value"` (line 343) ✅
- **StepName** — defaults to "Set Variable" (line 231) ✅
- **Clone** — InputVariableName preserved (line 1706) ✅
- **Execution service** — stores in `_readTextValues[varName]` (line 1468–1473) ✅

### ⚠️ Issue Found

#### No IsValid validation

- **What:** `MacroStepType.SetVariable` is **not checked** in the `IsValid` property. Falls through to default `return true`, meaning a SetVariable block with empty variable name or empty value is considered "valid"
- **Impact:** No red warning dot when the variable name is blank
- **Fix:** Add `if (Type == MacroStepType.SetVariable) return !string.IsNullOrWhiteSpace(InputVariableName);`

---

## Call Macro Block

### ✅ All Working

- **Value** (target macro name) — model, UI dropdown (LogicContainerTemplates.xaml line 234), compiler resolves by name (line 4090) ✅
- **Inline expansion** — compiler inlines the called macro's steps directly (line 4115: `CompileStepCollection`) ✅
- **Recursion protection** — call stack tracks depth, max 10 levels, detects loops (lines 4094–4107) ✅
- **Success tracking** — tracks `LastActionSucceeded` for Logic blocks (lines 4118–4130) ✅
- **Missing target detection** — `IsCallMacroTargetMissing()` shows warning if target macro was deleted (lines 1250, 1263, 1273) ✅
- **Rename sync** — `MacroDatabase.cs` updates CallMacro references when target is renamed (line 996) ✅
- **DisplayValue** — shows target macro name (line 347) ✅
- **IsValid** — requires non-empty Value (line 1113) ✅
- **StepName** — auto-generated unique name like "Macro 1" (line 193) ✅
- **Sandbox** — supported! Inline-expands called macro's steps with recursion protection (lines 4607–4625) ✅

### ✅ No Issues Found

---

## Delay (Wait Timer) Block

### ✅ All Working — No Issues

- **Duration** — model (line 450), clamped to `Math.Max(0)`, compiler scales by PlaybackSpeed with 10ms floor (line 1681) ✅
- **IsHumanized** — compiler uses `GetHumanizedDelay()` for random variance (line 1686) ✅
- **HumanizationLevel** — falls back to macro default when 0 (line 1684) ✅
- **DisplayValue** — shows HumanizedDuration e.g. "500ms" or "2000ms (2s)" (line 333) ✅
- **IsValid** — requires Duration > 0 (line 1073) ✅
- **Clone** — Duration, IsHumanized, HumanizationLevel all preserved ✅
- **Execution** — full parity with compiler, SafeDelay with token cancellation ✅
- **Init** — defaults to StepName="Wait", Duration=100ms ✅

---

## Mouse Trace Block

### ✅ All Working — No Issues

- **TraceFileId** — model (line 1137), compiler checks `FileExist()` before playback (line 2473) ✅
- **X/Y start position** — auto-populated from trace file via `EnsureTraceStartEndPoints()` (line 1169) ✅
- **Drag mode** — compiler detects via `Value.Contains("Drag")`, generates Click Down/Up with hold/release delays ✅
- **HoldDelayMs / ReleaseDelayMs** — model (lines 1154, 1161), compiler uses for drag timing ✅
- **CoordinateMode** — compiler sets/resets CoordMode, activates target window (lines 2459–2470) ✅
- **Physics profiles** — compiler applies Smooth/Linear via SmoothTraceEngine (lines 2417–2455) ✅
- **PlaybackSpeed** — compiler scales trace timing (lines 2488–2490) ✅
- **DisplayValue** — shows "Recorded trace" (line 401) ✅
- **IsValid** — requires non-empty TraceFileId (line 1110) ✅
- **Clone** — all 5 MouseTrace props preserved (lines 1738–1743) ✅
- **Execution** — full C# playback with window offsets and drag support (lines 1576–1637) ✅

---

## Rescan Findings (Second Pass)

### 🐛 NEW: Keyboard `IsHumanized` silently ignored

- **What:** The Keyboard block compiler (lines 1692–1734) **never checks `step.IsHumanized`**. Delay block checks it (line 1682), Mouse Drag checks it (line 1799), but Keyboard does NOT
- **Impact:** If user enables humanization on a Keyboard step, no randomized delay is added — silently ignored
- **Fix:** Add humanized delay before `Send()` in the Keyboard compiler, same pattern as Delay

### 🐛 NEW: Mouse Multi-Click/Timer Click always does Left Click

- **What:** At line 1810–1816, `clkParam` is only built when action is NOT "Multiple Clicks" or "Timer Click". For these two modes, `clkParam` stays empty `""`, so `Click()` always does Left Click
- **Impact:** If user sets Multiple Clicks or Timer Click with **Right/Middle/XButton**, the compiled AHK ignores the button — always clicks Left
- **Fix:** Build `clkParam` with the correct `buttonArg` for Multi/Timer click modes too

### ⚠️ NEW: Notification SoundType missing default fallback

- **What:** The SystemSound compiler (line 3796) has a `default:` fallback that plays `SoundPlay("*-1")`. The Notification sound compiler (lines 3745–3754) does NOT have a `default:` case
- **Impact:** If SoundType is somehow unexpected, Notification silently does nothing. Minor since UI constrains values

### ⚠️ NEW: SetVariable value escaping incomplete

- **What:** SetVariable compiler (line 2866) uses `step.Value?.Replace("\"", "\"\"")` — only escapes double quotes. Does NOT escape backticks (`` ` ``), `\n`, `\r`, `\t`
- **Impact:** A variable value containing a backtick would be interpreted as an AHK escape character (e.g., `` `n `` becomes a newline)
- **Fix:** Use `EscapeStringForAhkLiteral()` instead of manual `.Replace()`

---

## ImageSearch Block

### ✅ Mostly Working

- **SearchImageFilename** — model (line 1313), compiler uses with tolerance (line 2083) ✅
- **FindTextCode** (Fast engine) — model (line 1454), compiler uses with FindText() (line 2048) ✅
- **FindTextTolerance / FindTextBgTolerance** — model (lines 1432/1440), compiler uses both ✅
- **ImageTolerance** (Standard engine) — model (line 561), compiler clamps 0–255 (line 2084) ✅
- **UseFastEngine** — compiler switches between FindText() and ImageSearch() (line 2046) ✅
- **Scope parsing** — Full Screen, Smart Search, WIN_LIVE, WIN_REL all work ✅
- **SmartRetry** — Loop(maxRetries) with break-on-success and RetryDelayMs (lines 2029–2151) ✅
- **Offset** — OffsetX/OffsetY applied post-find (lines 2054–2055) ✅
- **DebugHighlight** — Red border overlay with 2s auto-destroy (lines 2129–2141) ✅
- **FailIfMissing** — checks IsStepFailureHandled for Logic-If fallback (lines 2154–2164) ✅
- **Window fallback** — 2-tier: small scope → full window retry (lines 2063–2078) ✅
- **StepSuccessStates** — tracks for Logic-If (lines 2167–2171) ✅
- **IsValid** — requires StepName + (FindTextCode or SearchImageFilename) (line 1075–1078) ✅
- **Clone** — SearchImageFilename, FindTextCode, FindTextTolerance, FindTextBgTolerance, ImageTolerance all preserved ✅

### 🐛 Issues Found

#### `WIN_SMART:` scope not handled in compiler

- **What:** Line 1927 only handles `WIN_LIVE:` and `WIN_REL:`. The `WIN_SMART:` scope falls through to the generic `else` branch (line 2006) which strips "WIN:" and splits by comma — this produces garbage coordinates
- **Impact:** Users who set a WIN_SMART scope on an ImageSearch step get broken search areas. **PixelSearch correctly handles WIN_SMART (line 2210)**
- **Fix:** Add `|| step.SearchScopeSummary.StartsWith("WIN_SMART:")` to the condition at line 1927, with full-window scope + fallback logic

#### `AutoLaunchPath` missing from Clone (shared)

- **What:** `AutoLaunchPath` (model line 588) is NOT included in the Clone method. Duplicating an ImageSearch step with a custom auto-launch path silently drops the path
- **Fix:** Add `AutoLaunchPath = this.AutoLaunchPath,` to Clone method

---

## PixelSearch Block

### ✅ All Working — Well Implemented

- **TargetColorHex** — model (line 544), compiler sanitizes and uses (line 2175) ✅
- **Tolerance** — model (line 553), compiler clamps 0–255 (line 2195) ✅
- **Scope parsing** — Full Screen, Smart Search, WIN_LIVE, WIN_REL, **WIN_SMART all handled** ✅
- **SmartRetry** — same Loop pattern as ImageSearch ✅
- **Self-healing fallback** — 2px expanded search if not found ✅
- **Offset** — OffsetX/OffsetY applied post-find ✅
- **DebugHighlight** — dot + ripple animation ✅
- **FailIfMissing** — with Logic-If awareness ✅
- **IsValid** — requires StepName + TargetColorHex + X/Y + SearchWidth/Height (line 1080) ✅
- **Clone** — all PixelSearch properties preserved ✅
- **Sandbox** — full preview support with scope parsing ✅

### ⚠️ Shared Issue Only

#### `AutoLaunchPath` missing from Clone (shared with all vision blocks)

---

## WindowAction Block

### ✅ All Working — Very Mature

- **Value** (action type) — Activate, Close, Minimize, Maximize, Restore, Move — all compiled (lines 2554–2801) ✅
- **Activate** — 5-step smart activation chain with WinWait, tab switching, process search, auto-launch ✅
- **Browser tab switching** — UIAutomation-based, compiled for AHK (lines 2573–2616) ✅
- **Move** — `WinMove(X, Y, W, H, title)` with CaptureWindowParameters (line 2775–2783) ✅
- **SmartWait** — WinWait with 10s timeout (lines 2656–2659) ✅
- **SearchOtherWindows** — WinGetList fallback (lines 2682–2710) ✅
- **AutoLaunchIfMissing** — Run() + WinWait() (lines 2722–2760) ✅
- **FailIfMissing** — return to abort (lines 2766–2772) ✅
- **Smart Desktop Click** — `Shell.Application.MinimizeAll()` (lines 2642–2652) ✅
- **IsValid** — requires StepName + Value + WindowTitle, CaptureWindowParameters needs WindowWidth > 0 ✅
- **Clone** — all window properties preserved (lines 1762–1768) ✅
- **Execution** — full C# preview with ShowWindow/SetForegroundWindow/MoveWindow ✅

### ⚠️ Shared Issue Only

#### `AutoLaunchPath` missing from Clone (shared with all vision blocks)

---

## UIElement Block

### ✅ Mostly Working — Very Complex

- **UIElementName / UIAutomationId / UIClassName / UIControlType** — model (lines 778–799), compiler builds AND conditions for multi-property matching ✅
- **UIWindowTitle / UIProcessName** — used for window targeting ✅
- **UIAction** — 11 actions compiled: Click, Double Click, Right Click, Set Text, Toggle, Check, Uncheck, Focus, Select, Check Exists, Wait Until Found/Gone, Read Text ✅
- **UIFindMode** (Exact/SameApp/FindLatest) — SameApp uses process matching ✅
- **UISetTextValue** — ValuePattern → Click+Type fallback ✅
- **UITimeoutSeconds** — timeout loop for wait actions ✅
- **UIFallbackToCoordinates** — physical click at element coordinates when patterns fail ✅
- **UsePhysicalClick** — Click() vs ControlClick toggle ✅
- **UIBackgroundMode** — ValuePattern/TogglePattern for background automation ✅
- **UIScrollIntoView** — ScrollItemPattern before action ✅
- **UIElementPath** — tree path for deep element targeting ✅
- **IsDebugHighlight** — cyan border overlay ✅
- **IsValid** — requires at least one of UIElementName/UIAutomationId/UIClassName (line 1112) ✅
- **Clone** — all 17 UIElement properties preserved (lines 1777–1794) ✅
- **Execution** — full native C# UIAutomation (lines 1802–2602) ✅

### 🐛 Issues Found

#### SameApp/UIMatchByProcess not compiled for AHK scripts

- **What:** Main compiler (line 3067) always uses `WinExist("windowTitle")` — never checks `UIMatchByProcess` or `UIFindMode == "SameApp"` to use `UIProcessName` with `ahk_exe`
- **Works in:** Sandbox preview (line 584) and execution service (ResolveWindowHandle) — but NOT in compiled AHK
- **Impact:** SameApp mode works in preview but fails in compiled scripts if window title changes
- **Fix:** Add SameApp check from SingleStep.cs before the WinExist() call in the main compiler

#### Click non-physical fallback missing else branch

- **What:** Line 3274–3290 — when InvokePattern fails and ControlClick checks `if (_WinHwnd)`, there's no `else` branch. If window handle is 0 (desktop root search), the click is silently dropped
- **Impact:** Rare — only desktop root elements where InvokePattern is unavailable
- **Fix:** Add `else { Click(_CX, _CY) }` fallback

---

## GroupHeader Block

### ✅ All Working — No Issues

- **ChildSteps** — recursively compiled inline (line 3853), deeply cloned (lines 1812–1818) ✅
- **GroupNote / GroupColor** — model (lines 1572/1579), cloned (lines 1774–1775) ✅
- **IsValid** — always true (organizer block has no required fields) ✅
- **Execution** — recursively processes ChildSteps ✅
- **Sandbox** — handled, iterates ChildSteps (lines 4602–4606) ✅

---

## LoopSequence Block

### ✅ Mostly Working

- **ClickCount** (loop count) — model (line 917), compiler wraps in `Loop N {}` (line 3807) ✅
- **ChildSteps** — compiled recursively inside Loop wrapper ✅
- **Token check** — injected inside loop for cancellation (lines 3809–3810) ✅
- **IsValid** — requires ClickCount > 0 (line 1109) ✅
- **Clone** — ClickCount + ChildSteps deeply cloned ✅
- **Execution** — loops `repeats` times with cancellation awareness (lines 722–732) ✅

### ⚠️ Issue Found

#### Sandbox ignores LoopSequence

- **What:** `CompileSandboxStep` (lines 4550–4629) has no case for LoopSequence — children inside loops are silently skipped in sandbox/hotkey test mode
- **Fix:** Add LoopSequence handling with `Loop N {}` wrapper in sandbox

---

## LogicIf / LogicElse / LogicEndIf Blocks

### ✅ Mostly Working

- **LogicMode** — 6 modes all compiled correctly:
  - NamedBlockSuccess → checks `StepSuccessStates` map ✅
  - NamedBlockFailed → inverted check ✅
  - AboveStepFailed → checks `!IsSet(LastActionSucceeded)` ✅
  - VariableEquals → checks `UserVariables` map ✅
  - VariableNotEquals → inverted check ✅
  - Default (AboveStepSuccess) → `LastActionSucceeded == 1` ✅
- **ChildSteps** (true branch) + **ChildStepsFalse** (else branch) — both compiled (lines 3855/3862–3868) ✅
- **IsSourceDisabled** — compiler emits comment + skips (line 3814) ✅
- **LogicSource** — used for NamedBlock modes ✅
- **LogicVariableName / LogicExpectedValue** — used for Variable modes ✅
- **Clone** — all logic properties + both child collections deeply cloned ✅
- **Execution** — all modes handled with recursive child processing (lines 662–706) ✅

### ⚠️ Issues Found

#### Sandbox ignores LogicIf

- **What:** `CompileSandboxStep` has no case for LogicIf — entire if/else logic and all their children are silently skipped in sandbox mode
- **Fix:** Add LogicIf handling with condition evaluation in sandbox

#### IsValid requires Value but compiler doesn't use it

- **What:** Line 1105 requires `!string.IsNullOrWhiteSpace(Value)` for all LogicIf modes. But the compiler never reads `Value` — it uses LogicMode, LogicSource, LogicVariableName, LogicExpectedValue. A fully configured LogicIf with empty `Value` shows a false warning dot
- **Impact:** UI annoyance — user sees red dot even though the block would work fine

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

### 🐛 NEW: FailIfMissing inside Loop kills entire macro

- **What:** Line 2162: `return` inside ImageSearch/PixelSearch FailIfMissing exits the entire `ExecuteMacro_N()` function. Inside a Loop, this kills the whole macro instead of just breaking the loop
- **Impact:** Users expect FailIfMissing inside a loop to skip/break, not abort everything
- **Fix:** Consider using `break` when inside a loop context

### ⚠️ NEW: ImageSearch path double-escapes backslashes (harmless)

- **What:** Line 2083: `.Replace("\\", "\\\\")` doubles all backslashes. AHK v2 doesn't use `\` as escape (backtick is escape). So `C:\\Users\\img.png` has literal double backslashes
- **Impact:** Works in practice because Windows normalizes redundant path separators, but it's unnecessary and confusing code
- **Note:** Same pattern used in Notification sound paths (line 3738) and WaitUntil (line 4019)

### ⚠️ Shared: AutoLaunchPath missing from Clone

- **What:** `AutoLaunchPath` (model line 588) not in Clone method — affects ImageSearch, PixelSearch, WindowAction, UIElement
- **Fix:** Single fix: add `AutoLaunchPath = this.AutoLaunchPath,` to Clone

---

## Cross-Block: ImageSearch + LogicIf + Click Flow

### ✅ Basic Flow Works

**Scenario:** ImageSearch "Button A" → LogicIf (NamedBlockSuccess) → Yes: Click / No: Search "Button B" + Click

- ImageSearch sets `LastActionSucceeded`, `StepSuccessStates["ButtonA"]`, and `FoundX/FoundY` on success ✅
- LogicIf checks `StepSuccessStates["ButtonA"] == 1` → branches correctly ✅
- Yes branch: Mouse Click with `ActionTarget = "Found Image"` uses `FoundX/FoundY` → clicks correct location ✅
- No branch: Second ImageSearch overwrites `FoundX/FoundY` with new coords → Click uses new location ✅
- Named targeting: Mouse Click with `ActionTarget = "Specific Image/Pixel"` and `LogicSource = "ButtonA"` uses `ButtonAX/ButtonAY` (step-specific vars) → independently targeted ✅

### ⚠️ Edge Case Found

#### Stale FoundX/FoundY on failure — no clearing

- **What:** When ImageSearch **fails**, `FoundX/FoundY` are NOT cleared — they keep their last successful values (initialized to `0,0` at macro start). So if ImageSearch 1 succeeds, then ImageSearch 2 fails, `FoundX/FoundY` still point to Image 1's location
- **Scenario:** Loop with ImageSearch → LogicIf → No branch has Mouse Click at "Found Image" → Click happens at **stale coordinates** from the last successful search, not the current failed one
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

### 🐛 NEW: WaitUntil doesn't set FoundX/FoundY

- **What:** WaitUntil (ImageFound) stores coordinates in `_WaitX/_WaitY` temp variables (lines 4015/4021), but NEVER copies them to `FoundX/FoundY` (lines 4060–4067). After WaitUntil succeeds, the found location is lost
- **Impact:** User does WaitUntil (ImageFound) → Mouse Click at "Found Image" — the click goes to stale coordinates (0,0 or last ImageSearch result), NOT where WaitUntil found the image
- **Compare:** ImageSearch explicitly sets `FoundX := {prefix}X` (lines 2057–2058). WaitUntil does NOT
- **Fix:** After line 4061 (`LastActionSucceeded := 1`), add `FoundX := _WaitX` and `FoundY := _WaitY`

### 🐛 NEW: Keyboard Hold modifier leaks into CallMacro

- **What:** Keyboard "Hold Down" (e.g., Shift) compiles to `Send("{Shift Down}")` (line 1733). CallMacro inlines steps directly (line 4115) with NO modifier reset. Any Mouse Click in the called macro becomes Shift+Click
- **Scenario:** Step 1: Hold Shift → Step 2: CallMacro "Click Macro" → Step 3: Release Shift. The click in Step 2 is Shift+Click unintentionally
- **Impact:** Called macro's clicks, types, and key presses all have the modifier applied — wrong behavior
- **Note:** Modifier cleanup only happens at macro END (line 1569) or on script exit (line 359), not between inlined steps

---

## Preview / Execution Service Deep Scan

All 23 blocks checked for preview vs compiler parity in `MacroExecutionService.cs`.

### ✅ Preview Matches Compiler — No Issues

| Block | Notes |
|---|---|
| Keyboard | Hold/Release, combos, all modes match ✅ |
| Text | Write/Paste, UseVariable all match ✅ |
| ImageSearch | Smart — preview compiles a mini AHK test script and runs it, same code path ✅ |
| PixelSearch | Same handler as ImageSearch, runs compiled test script ✅ |
| Popup | PopupMode missing in BOTH (known gap), otherwise matches ✅ |
| SetVariable | Perfect parity — same sanitization, same variable map ✅ |
| Delay | Full parity with humanization ✅ |
| MouseTrace | Full playback with physics engine ✅ |
| WindowAction | All 6 actions match ✅ |
| UIElement | Full native C# UIAutomation, all actions ✅ |
| GroupHeader / LoopSequence / LogicIf | Recursive child processing matches ✅ |
| CallMacro | Inline expansion with recursion protection ✅ |

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

#### MEDIUM: Notification ignores SoundType, Icon, and Timeout in preview

- **What:** Preview (line 1476) shows a modal `DarkMessageBoxWindow` with OK button. Ignores `step.SoundType` (no sound played), `step.NotificationIcon` (always shows Info), and `step.PopupTimeout` (blocks until user clicks OK instead of auto-dismissing)
- **Impact:** Preview notification is silent, wrong icon, and blocking — compiled version has sound, correct icon, and auto-timeout

#### LOW: SystemSound preview only plays 2 of 7 sound types

- **What:** Preview (line 1643) only handles "Exclamation", "Asterisk", "Custom File", and default Beep. Missing: Custom Beep (Short/Medium/Long/Double Alert), Success Chime, Error Chord — all fall back to generic system beep
- **Impact:** Most sounds play wrong sound in preview. Minor since user hears *something*

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

