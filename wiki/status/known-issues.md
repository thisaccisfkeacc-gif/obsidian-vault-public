---
tags: [status, bugs, issues, known]
date: 2026-08-01
sources:
  - raw/project-logs/VERSION_LOG.md
status: current
---

# Known Issues ðŸ›

## Previously Completed Audits âœ…
- **UI Element Block** â€” 42 bugs found & fixed (7 rounds)
- **Image Search Block** â€” 6 bugs found & fixed (Bugs 47-52)
- **Pixel Search Block** â€” 3 bugs found & fixed (Bugs 53-55)
- **Mouse Actions Block** â€” 6 bugs found & fixed (Bugs 56-61)
- **Conflict & Safety System** — 6 bugs found & fixed (Bugs 100-105, 2026-08-02)

## Active Findings — Conflict & Safety System

### All FIXED (2026-08-02)
- ~~**Bug 100 (ScriptLibraryView.xaml.cs):** Key capture stored letters as UPPERCASE (^A) but popup/hook showed lowercase (^a) — AHK is case-insensitive, so duplicate keys were never detected.~~ **FIXED** — Single-char keys normalized with ToLowerInvariant() at capture.
- ~~**Bug 101 (ScriptLibraryView.xaml.cs ~L445, L614):** Duplicate checks compared by card NAME instead of ID — renamed cards escaped detection.~~ **FIXED** — Now compares h.Id != bindingAction.Id.
- ~~**Bug 102 (ScriptLibraryViewModel.State.cs ValidateConflicts):** Conflict grouping was case-sensitive — Esc vs esc treated as different keys.~~ **FIXED** — GroupBy with StringComparer.OrdinalIgnoreCase + case-insensitive Equals in both bind paths.
- ~~**Bug 103 (ScriptLibraryViewModel.State.cs AreTriggersConflicting):** Two triggers in "Include app" mode with EMPTY app lists passed the no-overlap exemption (no conflict flagged), but compile without #HotIf scope → both become GLOBAL hotkeys → real runtime collision.~~ **FIXED** — Exemption now requires HasTargetApps on BOTH sides; empty lists = global = conflict.
- ~~**Bug 104 (ScriptLibraryView.xaml.cs):** Disabled cards blocked their key from being reused at bind time, while the validator ignores disabled cards — inconsistent.~~ **FIXED** — Both bind-time checks (keyboard + mouse) now require h.Enabled.
- ~~**Bug 105 (AppEnums.cs):** Dead TriggerMode.PressAndRelease enum value — hidden from UI, no compiler support.~~ **FIXED** — Removed; numeric values pinned (ScheduledTime = 10) so existing AppConfig.json saves stay compatible.

### Notes (Verified OK, no bug)
- Release + Press on same key coexist correctly (AHK UP variant).
- DoubleTap / Hold / LongPress / Toggle on same key all correctly flagged as conflicts.
- Global vs App-scoped same key correctly flagged.
- Compiler skips conflicting hotkeys; START gate blocks when conflicts exist.

## Active Findings â€” Keyboard / TypeText Block ðŸŽ¹

### ðŸ”´ HIGH Severity
- ~~**Bug 62 (ScriptCompilerService L1114):** Backtick (`` ` ``) not escaped in Keyboard `Send()` â€” AHK interprets it as escape char â†’ broken script.~~ **FIXED** â€” Added backtick to special char list + escaping in safeOutKey
- ~~**Bug 63 (MacroExecutionService L937):** Single special keys (Enter, Tab, F1, arrows, etc.) typed as literal text instead of pressed â€” `SafeSendKeys("Enter")` types E-n-t-e-r.~~ **FIXED** â€” Uses `SendKey()` with `GetVirtualKeyCode()` for recognized keys
- ~~**Bug 64 (MacroExecutionService L34):** `keybd_event` P/Invoke `dwExtraInfo` declared as `int` instead of `UIntPtr` â€” wrong size on 64-bit systems.~~ **FIXED** â€” Changed to `UIntPtr`, all call sites updated
- ~~**Bug 65 (MacroExecutionService L100):** `KEYEVENTF_EXTENDEDKEY` defined but never used â€” extended keys may be misinterpreted.~~ **FIXED** â€” Added `_extendedKeys` set + `SendKey()` helper that auto-applies flag

### ðŸŸ¡ MEDIUM Severity
- ~~**Bug 66 (MacroExecutionService L869):** Scan code always 0 in `keybd_event` â€” keys silently dropped in games, DirectInput, RDP, VMware.~~ **FIXED** â€” `SendKey()` uses `MapVirtualKey()` for scan codes
- ~~**Bug 67 (ScriptCompilerService L1715):** Fast Paste uses fixed `Sleep(50)` instead of `ClipWait` â€” may paste empty on slow systems.~~ **FIXED** â€” Replaced with `ClipWait(1)`
- ~~**Bug 68 (ScriptCompilerService L3542):** Sandbox compiler ignores `KeyActionType` — Hold/Release broken in test mode.~~ **FIXED 2026-06-16** — Sandbox keyboard now checks `KeyActionType` and emits `{Key Down}` / `{Key Up}` correctly

### ðŸŸ¢ LOW Severity
- **Bug 69 (ScriptCompilerService L1725):** `SendEvent("{Text}")` should be `SendText()` â€” works but fragile. **FOUND**
- **Bug 70 (ScriptCompilerService L1722):** No humanization for typing speed â€” always fixed 50ms/key. **FOUND**
- **Bug 71 (MacroExecutionService L1011):** Unicode/emoji not supported in char-by-char TypeText via SendKeys. **FOUND**

## Active Findings â€” Window Actions Block ðŸªŸ

### ðŸ”´ HIGH Severity
- ~~**Bug 72 (ScriptCompilerService L1811):** `WinWait` timeout crashes macro â€” AHK v2 throws `TimeoutError` with no try/catch wrapper.~~ **FIXED** â€” Wrapped in try/catch
- ~~**Bug 73 (ScriptCompilerService L1729-1926):** `StepSuccessStates` never set for WindowAction â€” Logic If blocks can't branch on results.~~ **FIXED** â€” Added `StepSuccessStates[label] := LastActionSucceeded`
- ~~**Bug 74 (MacroExecutionService L1440):** Always returns `true` regardless of outcome â€” breaks LogicIf/AboveStepFailed.~~ **FIXED** â€” Sets `_lastActionSucceeded = false` when window not found
- ~~**Bug 75 (MacroExecutionService L1259-1440):** `SmartWait` completely ignored in C# executor — no WinWait equivalent, no polling.~~ **FIXED 2026-08-01** — Implemented: AHK compile emits `WinWait(title, , 10)` when `step.SmartWait` (ScriptCompilerService.cs:2948-2951); C# executor polls window restoration (MacroExecutionService.cs:2056-2083)

### ðŸŸ¡ MEDIUM Severity
- ~~**Bug 76 (ScriptCompilerService L1731):** Window title not fully escaped â€” missing backtick/newline escaping.~~ **FIXED** â€” Uses `EscapeStringForAhkLiteral()`
- **Bug 77 (ScriptCompilerService L1877-1895):** AutoLaunch doesn't wait for window to appear. **UNFIXED** (feature gap)
- ~~**Bug 78 (MacroExecutionService L1433):** `?.Dispatcher.Invoke` missing second `?.` â€” potential NRE.~~ **FIXED** â€” Added `?.Dispatcher?.Invoke`

### ðŸŸ¢ LOW Severity
- **Bug 79 (ScriptCompilerService L1744):** Double-escape on exePart/titlePart â€” quotes in window titles get quadruple-escaped. **FOUND**
- **Bug 80 (ScriptCompilerService L1804):** No DPI awareness for WinMove coordinates. **FOUND**
- **Bug 81 (ScriptCompilerService L1911):** WinClose has no WinKill fallback for hung apps. **FOUND**

## Active Findings â€” Logic Blocks ðŸ§ 

### ðŸŸ¡ MEDIUM Severity
- ~~**Bug 82 (ScriptCompilerService L2895):** Variable value injection in LogicIf â€” `LogicExpectedValue` only escapes `"`, not backticks or AHK special chars.~~ **FIXED** â€” Uses `EscapeStringForAhkLiteral()`
- **Bug 83 (MacroExecutionService L413-462):** `IsSourceDisabled` not checked in C# executor LogicIf â€” evaluates condition with stale/default data instead of skipping. **FOUND** (re-verified 2026-08-01 — LogicIf executor at MacroExecutionService.cs:734-767 checks only `step.IsDisabled` at line 731, not `IsSourceDisabled`)

### ðŸŸ¢ LOW Severity
- **Bug 84 (ScriptCompilerService L2904):** Empty LogicSource on NamedBlockSuccess/Failed silently falls back to AboveStepSuccess condition â€” confusing for users. **FOUND**

## Active Findings â€” Remaining Blocks ðŸ“¦

### ðŸ”´ HIGH Severity
- **Bug 85 (MacroExecutionService L1193):** UserInput stores variable under `StepName` instead of `InputVariableName` — regression/incomplete fix in C# execution path. **FOUND**
- ~~**Bug 86 (MacroExecutionService):** `WaitUntil` block completely missing from C# executor — silently skipped, returns true. Macros depending on it click wrong targets.~~ **FIXED 2026-06-16** — Added `WindowActive` polling loop; ImageFound/PixelFound pass-through with note that AHK handles them
- ~~**Bug 87 (MacroExecutionService):** `WaitForKey` block completely missing — silently skipped, macro continues without pausing.~~ **FIXED 2026-06-16** — Added `MessageBox.Show()` OK/Cancel dialog; Cancel aborts macro

### ðŸŸ¡ MEDIUM Severity
- ~~**Bug 88 (MacroExecutionService):** `FileLauncher` block missing from C# executor — silently skipped.~~ **FIXED 2026-06-16** — Added `ProcessStartInfo(step.Value) { UseShellExecute = true }` launch
- ~~**Bug 89 (MacroExecutionService L1236):** Sound type strings don't match AHK compiler — "Custom Beep (Short)", "Success Chime" etc. play default beep instead.~~ **FIXED 2026-06-16** — Expanded switch to map all 7 AHK sound type names to correct system sounds

## Trigger / Hotkey Compilation ðŸ”‘

### ðŸŸ¢ No Active Bugs

### Notes (By Design)
- **Note 1 (KeyCaptureWindow L127-161):** In advanced combo mode, single key capture happens on key **release** (not press) because the system waits to see if a second key is coming. This adds a slight perceived delay for single keys when the toggle is ON. **BY DESIGN**


## Text Snippets 📝

### 🔴 HIGH Severity
- ~~**Bug 90 (ScriptCompilerService L3294):** `SoundPlay` throws a fatal AutoHotkey v2 error and stops script execution if audio devices are unavailable/busy.~~ **FIXED** — Wrapped in `try { SoundPlay(...) } catch {}` block.
- ~~**Bug 91 (ScriptCompilerService):** Inline comments after #HotIf directives (e.g. #HotIf ; Global Scope) get incorrectly evaluated by AutoHotkey v2 as invalid expressions, causing an "Error: Invalid usage" crash.~~ **FIXED** — Removed comments to ensure clean #HotIf directives.

## Smart View 

### Open Bugs
- **Bug (SmartView.cs L559-563):** Window blocks get absorbed into keyboard combo groups in Smart View. Example: Recording Ctrl+C -> Ctrl+Shift+V -> Wait -> Window:Ditto shows the Window block in Raw View but it disappears in Smart View. Root cause: modifier wrap loop's catch-all tolerates non-keyboard/delay steps, swallowing Window blocks into `currentChunk`. **Status: Needs investigation** (reported 2026-06-14)

## Mouse Block Preview — Button Does Nothing 🖱️

### 🔴 HIGH Severity
- **Bug 92 (MacroEditorViewModel.Commands.cs L426+):** `UnifiedPreviewCommand` has no `else if` branch for `MacroStepType.MouseClick` or `MacroStepType.MouseTrace` — the step silently falls through all conditions and nothing happens. Fix: add a branch that routes to `PreviewSonarCommand` (sonar pulse at X/Y). **FOUND** (re-verified 2026-08-01 — branches cover UIElement, WindowAction, ImageSearch/PixelSearch/WaitUntil, SystemSound, Popup, Notification; no MouseClick/MouseTrace branch)

### 🟡 MEDIUM Severity
- **Bug 93 (MacroEditorView.xaml, context menu ~L1312-1392):** Right-click "Preview Step" menu item has visibility triggers for ImageSearch, PixelSearch, UIElement, WindowAction, SystemSound, Popup, Notification, FileLauncher — but `MouseClick` is missing, so the option is invisible for mouse blocks. **FOUND** (re-verified 2026-08-01 — "Preview Step" item at MacroEditorView.xaml:1387, triggers through line 1497 cover ImageSearch/PixelSearch/UIElement/WindowAction/SystemSound/Popup/Notification/FileLauncher/UserInput/WaitForKey; no MouseClick/MouseTrace)

## Compact Blocks 🧱

### 🔴 HIGH Severity
- ~~**Bug 94 (MacroStepCard.xaml L267):** `WaitUntilPanelInlineTemplate` is referenced via DynamicResource but **never defined** in any template file — WaitUntil blocks show BLANK in compact mode.~~ **FIXED 2026-08-01** — Template now defined at SearchTemplates.xaml:2049 (verified)

## Kill Switch 🔑

### 🟡 MEDIUM Severity
- ~~**Bug 95 (MacroExecutionService.cs L230-253):** Preview kill switch is hardcoded to Double Escape only — ignores user's `MasterKillSwitchKey` setting from dropdown.~~ **FIXED 2026-08-01** — Kill key now resolved from `MasterKillSwitchKey` setting (MacroExecutionService.cs:246, 270, 3496) with mapping fallback to "Shift + Escape"
- ~~**Bug 96 (AppConfig.cs L88):** Dead legacy property `MasterKillSwitch` exists alongside active `MasterKillSwitchKey` (L92) — never read by any logic.~~ **FIXED 2026-08-01** — Legacy `MasterKillSwitch` property removed; only `MasterKillSwitchKey` remains (AppConfig.cs:100), used by executor, recorder, compiler and UI (verified)

## Context Menu ✏️

### 🟡 MEDIUM Severity
- ~~**Bug 97:** Right-click → Rename opens a dedicated window but clicking OK does NOT update the block name.~~ **FIXED 2026-08-01** — `EditStepNameCommand` now applies the new name (Commands.cs:1327-1389): updates `StepName`, syncs Group/Loop `Value`, rewrites LogicSource references, sets dirty flag (verified)

## UI / Visual 🎨

### 🟢 LOW Severity
- ~~**Bug 98:** WPF Focus Border (thin black rectangle) appears on buttons when clicked — looks ugly on dark theme. Fix: `FocusVisualStyle="{x:Null}"` app-wide.~~ **FIXED 2026-08-01** — Global empty FocusVisualStyle template at App.xaml:57 (verified)
- ~~**Bug 99 (App.xaml L66, MacroEditorView.xaml L1311):** Global White-Menu flash — deleting a step causes context menu to flash white for ~500ms.~~ **FIXED 2026-08-01** — Implicit dark ContextMenu style added at App.xaml:94 (`<Style TargetType="ContextMenu" BasedOn="{StaticResource DefaultContextMenuStyle}"/>`) so ALL context menus use the dark theme (verified)
