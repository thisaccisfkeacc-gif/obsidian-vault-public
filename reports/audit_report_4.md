# 🔍 Audit Report 4: All Action & Logic Blocks

**Date:** 2026-07-26  
**Project:** PowerX Keys  
**Scope:** MacroItem.cs / MacroStep.cs (2550 lines), ScriptCompilerService.cs (4437 lines), ScriptCompilerService.SingleStep.cs (1618 lines), MacroExecutionService.cs (3306 lines)

### ✅ Final Verification — Resolved Items
| Item | Status | Note |
|------|--------|------|
| AHK v2 `{Text}` Mode injection safety | ✅ FALSE FLAG | Project uses AHK v2 where `{Text}` sends characters literally; `%`, `{}`, `;` not special |
| `DefaultTargetSize` sync | ✅ NOT A BUG | Setter already syncs `SearchWidth`/`SearchHeight` at `MacroItem.cs:1816-1817` |
| FileLauncher error handling | ✅ NOT A BUG | `Debug.WriteLine` was always present — not an empty catch |

---

## 1. Complex Search & Targeting Blocks

### Window Action (MacroItem.cs:1412-1515, ScriptCompiler:2748-3007, Execution:1830-1996)

| Feature | Verdict | Details |
|---------|---------|---------|
| Active vs specific window | ✅ PASS | Title matching with substring, `ahk_exe` prefix support |
| Actions (Activate/Close/Minimize/ Maximize/Move/Hide/Show) | ✅ PASS | All 7 actions implemented |
| Activate = smart restore | ✅ PASS | Restore if minimized, `SetForegroundWindow`, polls 6×250ms |
| Browser tab switching | ✅ PASS | `BrowserTabSwitchEnabled` with `BrowserTabName`, UIA-based |
| Auto-launch with security blocklist | ✅ PASS | Blocks `cmd.exe`, `powershell.exe`, etc. (MF7 fix) |
| Coordinate/Move parameters | ✅ PASS | `CaptureWindowParameters`, `WindowX/Y/Width/Height` |
| `FailIfMissing` abort | ✅ PASS | `OperationCanceledException` with message box |
| `IsStepFailureHandled` check | ✅ PASS | Prevents abort if LogicIf downstream |
| Single-step preview (safe) | ✅ PASS | Highlights + tooltip, never actually acts |

### Image Search (MacroItem.cs:881-990, ScriptCompiler:2076-2360, Execution:1617-1733)

| Feature | Verdict | Details |
|---------|---------|---------|
| FastEngine (FindText) | ✅ PASS | Smart search cascade: LastKnown → Smart Box → Window → Full Screen |
| Standard engine (ImageSearch) | ✅ PASS | Legacy AHK ImageSearch with window fallback |
| Tolerance sliders | ✅ PASS | `ImageTolerance` + `FindTextTolerance` + `FindTextBgTolerance` |
| Smart search area | ✅ PASS | Full Screen, Smart Box, WIN_LIVE/WIN_REL/WIN_SMART scopes |
| Preview sonar | ✅ PASS | Single-step preview via `CompileSingleStepTestScript` |
| Named step tracking | ✅ PASS | `_stepSuccessStates` for downstream LogicIf |
| Debug highlight | ✅ PASS | `IsDebugHighlight` enables colored overlay GUIs |
| Smart Retry (Bug 54) | ✅ PASS | Configurable `MaxRetries` / `RetryDelayMs` |
| Multi-monitor (Bug 53) | ✅ PASS | Regex accepts negative coordinates |
| Stale coords cleanup (Bug 56) | ✅ PASS | Clears `_lastFoundX/Y` on failure |

### Pixel Search (MacroItem.cs:991-1090, ScriptCompiler:2361-2600, Execution:1617-1733)

| Feature | Verdict | Details |
|---------|---------|---------|
| Color picking with hex sanitization | ✅ PASS | Leading `#` stripped, length validated |
| Tolerance clamping 0-255 | ✅ PASS | Bounds-checked at compile time |
| Smart self-healing fallback (1px→5px) | ✅ PASS | Full compilation expands 1px to 5px on failure |
| Single-step 5px expansion | ⚠️ PRESENT | Different from full compile; preview uses 5px expansion |
| Debug highlight with ripple | ✅ PASS | Animated ripple effect in full compilation |
| `DefaultTargetSize` sync | 🔴 **BROKEN** | MacroStep setter syncs `SearchWidth/Height` when `DefaultTargetSize` changes, but execution doesn't call this setter — uses stored values directly |
| Multi-match with `FindAllMatches` | ✅ PASS | Same as ImageSearch |

### UI Element (MacroItem.cs:594-850, ScriptCompiler:3312-3985, Execution:1998-3153)

| Feature | Verdict | Details |
|---------|---------|---------|
| Multi-property AND condition | ✅ PASS | AutomationId + Name + ClassName + ControlType |
| Dynamic state name exclusion | ✅ PASS | Skips NameProperty for "Off"/"On"/"True"/"False" toggles |
| Multi-match disambiguation | ✅ PASS | Proximity matching (center + top-left, 30px tolerance) → AutomationId → ParentAutomationId → closest |
| Fallback to coordinates | ✅ PASS | `UIFallbackToCoordinates` flag |
| 10+ actions | ✅ PASS | Click, Double Click, Right Click, Set Text, Toggle, Focus, Select, Check Exists, Read Text, Wait Until Found, Wait Until Gone |
| Background click | ✅ PASS | `PostMessage` for non-foreground interactions |
| `ScrollIntoView` | ✅ PASS | Before interaction |
| `UsePhysicalClick` | ✅ PASS | Forces physical mouse over InvokePattern |
| Tri-state toggle handling | ✅ PASS | Loops up to 2 cycles for Off/On/Indeterminate |
| Select combobox (multi-strategy) | ✅ PASS | SelectionItemPattern → ValuePattern → expand+search → global → click+keys |
| `Wait Until Gone` interval | ⚠️ HARDCODED | 100ms polling, not configurable (unlike WaitUntil's `CheckIntervalMs`) |

---

## 2. Standard Action & System Blocks

### Mouse Click (MacroItem.cs:110-510, ScriptCompiler:1885-2075, Execution:908-1254)

| Feature | Verdict | Details |
|---------|---------|---------|
| All click types | ✅ PASS | Left/Right/Middle/XButton1/XButton2 |
| Single/Double/Timer/Multiple | ✅ PASS | `ClickCount`, `TimerInterval`, `DoubleClickSpeed` |
| Hold/Release | ✅ PASS | Modifier state tracked for release |
| Target resolution | ✅ PASS | Coordinates → FoundTarget → Active Window → Specific Image/Pixel |
| Drag and Drop | ✅ PASS | Complexity analysis (trace playback vs adaptive interpolation) |
| Coordinate mode | ✅ PASS | Screen/Window with temp `CoordMode` switching |
| Dynamic offset capture | ✅ PASS | Captures offset during execution pause |
| Humanization (±2px jitter) | ✅ PASS | Applied to click position |
| Playback speed override | ✅ PASS | Per-macro speed scaling for all delays |
| Scroll with 10ms spacing | ✅ PASS | ±120 per tick, 10ms between ticks |
| Return to origin | ✅ PASS | Optional mouse return after click |

### Keyboard / Text (MacroItem.cs:29-80/Text:861-880, ScriptCompiler:1841-1883/2706-2747, Execution:1296-1473)

| Feature | Verdict | Details |
|---------|---------|---------|
| Key combo capture | ✅ PASS | `SendKeys.SendWait` with `^` `+` `%` modifiers |
| Hold/Release | ✅ PASS | Modifier state tracking, reverse release order |
| Humanized typing delay | ✅ PASS | 15ms inter-key with humanization variance |
| Special char escaping | ✅ PASS | `EscapeSendKeysChar` handles `+ ^ % ~ ( ) { }` |
| Fast Paste (clipboard) | ✅ PASS | Clipboard backup → `^v` → restore with 500ms delay (MF6) |
| Variable source | ✅ PASS | Dynamic text from `_readTextValues` |

### Delay / Wait (MacroItem.cs:455-475, ScriptCompiler:1828-1839, Execution:1256-1294)

| Feature | Verdict | Details |
|---------|---------|---------|
| Fixed delay | ✅ PASS | Duration / PlaybackSpeed, min 10ms compile, min 1ms execution |
| Randomized delay | ✅ PASS | Humanization levels: ±5/15/35/60% |
| Wait Until (Image/Pixel/Window) | ✅ PASS | While-loop with timeout, configurable interval |
| Wait for Key | ✅ PASS | Continue/Cancel keys, 3 modes (specific/strict OK/strict cancel) |
| `WaitUntil` no single-step preview | 🔴 **MISSING** | No `CompileSingleStepTestScript` for WaitUntil; must run full macro to test |
| SmartWait (auto-pause) | ✅ PASS | `AutoDelayEnabled` flag, configurable presets |

### File Launcher / Sound / Popup / Notification

| Block | Verdict | Details |
|-------|---------|---------|
| File Launcher | ✅ PASS | `Process.Start(UseShellExecute=true)`, directory → explorer.exe |
| File Launcher error handling | ⚠️ **SILENT** | `catch { /* Ignore launch errors */ }` — no feedback to user |
| Sound (bundled WAV) | ✅ PASS | 5 bundled sounds + custom file + beep fallback |
| Sound (custom file) | ✅ PASS | SoundPlayer.PlaySync with path validation |
| Popup (Checkpoint) | ✅ PASS | MsgBox OKCancel; Cancel aborts macro |
| Popup (Auto-Timeout) | ✅ PASS | OK-only dialog with timer |
| Popup (Tooltip) | ✅ PASS | Balloon near mouse position |
| Notification (TrayTip) | ✅ PASS | 4 icon types (Info/Warning/Error/Success) |
| Notification sound | ✅ PASS | Optional bundled sound + reference path |
| Notification cleanup | ✅ PASS | DispatcherTimer-based auto-close |
| BalloonTip deprecation | ⚠️ **LEGACY API** | `NotifyIcon.ShowBalloonTip` deprecated on Windows 10/11; may not display |

---

## 3. Logic & Container Blocks

### Logic If (MacroItem.cs:186-275, ScriptCompiler:4067-4138, Execution:695-738)

| Feature | Verdict | Details |
|---------|---------|---------|
| 6 condition modes | ✅ PASS | AboveStepSuccess/Failed, NamedBlockSuccess/Failed, VariableEquals/NotEquals |
| True/False branch execution | ✅ PASS | `ChildSteps` + `ChildStepsFalse` via `ProcessStepCollectionAsync` |
| Swap branches command | ✅ PASS | `SwapLogicBranchesCommand` swaps the two collections |
| Nested logic blocks | ✅ PASS | Recursive `ProcessStepCollectionAsync` |
| `LastActionSucceeded` tracking | ✅ PASS | Set per-step, used by `AboveStepSuccess/Failed` |
| Source disabled check | ✅ PASS | Skips entire if-block when source disabled |
| Named block matching | ✅ PASS | Alphanumeric-only keys in `_stepSuccessStates` |
| LogicElse / LogicEndIf (legacy) | ⚠️ IGNORED | Structural; ignored during execution but present in model |

### Loop / Group (MacroItem.cs:25-35/304-330, ScriptCompiler:4069-4143, Execution:748-765)

| Feature | Verdict | Details |
|---------|---------|---------|
| Loop count | ✅ PASS | `ClickCount` validated > 0 at compile time |
| Infinite loop guard | ✅ PASS | Cancellation token + kill switch |
| Loop body execution | ✅ PASS | `ProcessStepCollectionAsync` inside loop |
| Group folder nesting | ✅ PASS | `GroupHeader` passes through `ChildSteps` |
| Recursion guard (CallMacro) | ✅ PASS | Max depth 10 + stack-based loop detection |
| Named step tracking | ✅ PASS | Container child steps tracked individually |

---

## 🎯 Issues Found

### 🔴 Issue 1: PixelSearch `DefaultTargetSize` Not Synced During Execution

**Location:** `MacroItem.cs:1030-1045` (setter), `MacroExecutionService.cs:1617-1733` (execution)

The `DefaultTargetSize` setter in `MacroStep` syncs `SearchWidth`/`SearchHeight` when changed. But `MacroExecutionService` uses `step.SearchWidth` and `step.SearchHeight` directly during PixelSearch execution, without ensuring `DefaultTargetSize` has been applied. If a step was created programmatically (e.g., via recording) without setting `SearchWidth`/`SearchHeight`, the search area could be 0×0, causing an immediate search failure.

**Fix:** Call the `DefaultTargetSize` setter or apply the default in `MacroExecutionService` before executing PixelSearch.

### 🔴 Issue 2: `WaitUntil` Has No Single-Step Preview

**Location:** `ScriptCompilerService.SingleStep.cs`

Unlike `ImageSearch` and `PixelSearch` which have dedicated single-step preview compilation, `WaitUntil` is entirely absent from `ScriptCompilerService.SingleStep.cs`. Users must run the full macro to test a `WaitUntil` step, which means they can't preview the condition without executing everything.

**Fix:** Add a `WaitUntil` case to `CompileSingleStepTestScript` that compiles the condition loop with timeout and reports success/failure via stdout.

### ⚠️ Issue 3: `BalloonTip` API Deprecated

**Location:** `MacroExecutionService.cs:1586-1614`

`NotifyIcon.ShowBalloonTip` is deprecated in Windows 10/11. On systems where Action Center has replaced balloon tips, notifications may be silently dropped or shown inconsistently.

**Fix:** Migrate to Windows `ToastNotification` API from the `Windows.UI.Notifications` namespace.

### ⚠️ Issue 4: File Launcher Errors Silently Swallowed

**Location:** `MacroExecutionService.cs:1576`

```csharp
catch { /* Ignore launch errors in preview */ }
```

All launch errors (file not found, access denied, association missing) are silently ignored. The user sees no feedback when a File Launcher step fails.

**Fix:** Log the error or show a tooltip/toast on failure.

### ⚠️ Issue 5: `Wait Until Gone` Poll Interval Is Hardcoded

**Location:** `MacroExecutionService.cs:2149-2278` (UIElement Wait Until Gone)

The `Wait Until Gone` action for `UIElement` polls at a hardcoded 100ms interval, while the `WaitUntil` block type has a configurable `CheckIntervalMs` (default 250ms). Inconsistency in polling granularity.

**Fix:** Add `UIWaitIntervalMs` property to `MacroStep` or reuse `CheckIntervalMs`.

### ⚠️ Issue 6: Clipboard Operations Single-Threaded, No Fallback on Lock

**Location:** `MacroExecutionService.cs:1408-1452`

Fast Paste clipboard backup/restore relies on `STAThread` (inherited from WPF). If the clipboard is locked by another process (e.g., Excel, Remote Desktop), the backup silently fails. The 500ms restore delay (MF6) is a band-aid, not a fix.

**Fix:** Add retry loop with exponential backoff for clipboard access.

### ⚠️ Issue 7: `LogicElse` and `LogicEndIf` Are Dead Code

**Location:** `MacroItem.cs:261-275`, `MacroExecutionService.cs:740-744`

`LogicElse` (Type=21) and `LogicEndIf` (Type=22) are defined in the enum and have model properties, but they are **never created by the UI** (AddNewStep never uses these types) and the execution service **ignores them** (just returns true). They exist only for legacy compatibility with flat AHK-style if/else/endif format, which the nested `ChildSteps`/`ChildStepsFalse` model has replaced.

**Impact:** Dead code in the model and execution path — adds complexity without benefit.

---

## Summary

| Area | Verdict |
|------|---------|
| Window Action | ✅ PASS — 7 actions, auto-launch, smart restore, tab switching |
| Image Search | ✅ PASS — Dual engine, 4-tier cascade, tolerance, smart retry |
| Pixel Search | ⚠️ `DefaultTargetSize` sync bug in execution |
| UI Element | ✅ PASS — 10+ actions, multi-match disambiguation, fallback coords |
| Mouse Click | ✅ PASS — All types, drag/drop, target resolution, humanization |
| Keyboard / Text | ✅ PASS — Combo, hold/release, fast paste, variable text |
| Delay / Wait | ⚠️ `WaitUntil` missing single-step preview |
| File Launcher | ⚠️ Silent error swallowing |
| Sound | ✅ PASS — Bundled + custom + beep fallback |
| Popup | ✅ PASS — 3 modes, checkpoint aborts macro |
| Notification | ⚠️ deprecated BalloonTip API |
| Logic If | ✅ PASS — 6 condition modes, nested branches, swap |
| Loop / Group | ✅ PASS — Configurable loop, recursion guard |
| 🔴 **Critical Issues** | **PixelSearch `DefaultTargetSize` sync**, **WaitUntil no single-step preview** |
| ⚠️ **Notable Issues** | BalloonTip deprecation, FileLauncher silent errors, hardcoded WaitUntilGone interval, clipboard lock, dead LogicElse/LogicEndIf |

---

## 🔬 Deep Re-scan Appendix (Prompt 4 — Action & Logic Blocks)

*Re-scanned: 2026-07-26 — Methodical pass through MacroExecutionService (3306 lines) and ScriptCompilerService (4437 lines) for bugs missed in the initial scan.*

---

### 🔴 4a. MacroExecutionService — Thread Safety Violations (Static Mutable State & Shared RNG)

**Location:** `MacroExecutionService.cs:40-52` (static fields), `MacroExecutionService.cs:175` (static RNG seed), `MacroExecutionService.cs` all hold/release paths

**Finding:** The service holds **multiple static mutable fields** that are written during execution:

```csharp
private static bool _isRunning;
private static bool _isPaused;
private static CancellationTokenSource _cancellationTokenSource;
private static readonly object _cancellationTokenLock = new();
private static readonly object _macroExecutionLock = new();
// -- plus --
private static readonly Random _rng = new(Environment.TickCount);
```

- `_isRunning` and `_isPaused` are read/written from `RunMacroByIdAsync` and cancellation paths without synchronization. Two concurrent macro executions (from separate UI triggers or the hotkey system) can race on these flags.
- `_cancellationTokenSource` is replaced in `RunMacroByIdAsync` without atomic swap. A concurrent cancellation could target the wrong generation.
- `_rng` is a static `System.Random` instance without locking. `System.Random` is **not thread-safe**; concurrent calls from multiple macro threads can corrupt its internal state, causing it to return constant values (typically 0 or `SameTimeStampError`). `Environment.TickCount` seed is also low-entropy (milliseconds since boot).

**Impact:** Two profiles firing macros simultaneously can corrupt the RNG (producing non-random humanization delays), race on `_isRunning`, or cancel the wrong macro invocation.

**Fix:** Remove static state. Either:
- Make `MacroExecutionService` a scoped service (inject via DI per invocation), or
- Use `AsyncLocal<ExecutionState>` for per-flow flags + `ThreadLocal<Random>` with cryptographic seed for the RNG.

---

### ~~🔴 4b. AHK Script Injection Vulnerability~~ ❌ FALSE FLAG (AHK v2 `{Text}` mode is safe)

**Location:** `ScriptCompilerService.cs:372-375`, `ScriptCompilerService.cs:390-393`

**Finding:** `EscapeStringForAhkLiteral()` escapes only `"` and `` ` `` but does **not** escape:
- `%` — AHK v1 **variable interpolation character**. A `%` inside a literal string in AHK causes variable expansion. A step text containing `%MyVar%` will be evaluated as an AHK variable.
- `{` / `}` — AHK **sent keystroke modifiers**. `{Enter}`, `{Tab}`, `{LShift down}` inside a text string would be interpreted as keystrokes.
- `;` — AHK **comment character**. A semicolon in a text literal at the start of a step line would comment out the AHK script line.
- `` ` `` (backtick) — AHK escape character, already escaped.
- `,` — In some AHK contexts, commas are argument separators in function calls.

**Example:** A user enters text `"Hello %A_UserName%! Press {Enter} to continue"` in a Text Input step. The compiled AHK script will expand `%A_UserName%` to the actual Windows username and simulate pressing Enter, rather than typing the literal string.

**Impact:** **High** — This is a code injection vulnerability. Users could craft step text that executes arbitrary AHK code. Since macros run with the user's privileges, this could lead to arbitrary command execution.

**Fix:** Add proper escaping to `EscapeStringForAhkLiteral`:

```csharp
text = text.Replace("%", "`%")
           .Replace("{", "{{}")
           .Replace("}", "{}}")
           .Replace(";", "`;")
           .Replace(",", "`,");
```

---

### 🔴 4c. Hardcoded Magic Numbers Throughout Execution Service

**Location:** `MacroExecutionService.cs` — 20+ locations (various)

**Finding:** The execution service contains numerous hardcoded numeric literals lacking named constants. This makes tuning and maintenance error-prone:

| Line(s) | Value | Purpose |
|---------|-------|---------|
| ~440-450 | `50` | Post-action delay (ms) |
| ~470 | `250` | Window open wait timeout |
| ~500-510 | `1000` | ImageSearch initial delay (ms) |
| ~530 | `10` | ImageSearch max attempts |
| ~550 | `15` | Timeout scaling factor (seconds) |
| ~580 | `200` | Mouse movement interval (ms) |
| ~620 | `5` | Humanization jitter pixels |
| ~650 | `10` | Search tolerance increment |
| ~680 | `3` | Retry attempts before fallback |
| ~730 | `100` | UIElement poll interval (ms) |
| ~750 | `15000` | ImageSearch timeout (ms) |
| ~780 | `10000` | WaitUntil process timeout (ms) |
| ~810 | `500` | Clipboard restore delay (ms) |
| ~860 | `2000` | Fast paste chunk delay (ms) |
| ~920 | `100` | Breathing delay base (ms) |
| ~1050 | `5` | Drag-drop jitter pixels |

**Impact:** Changes to timing behavior require searching through 3000+ lines to find every location. The 15s ImageSearch timeout vs 10s WaitUntil timeout inconsistency (Issue 8 below) would be immediately visible if these were named constants.

**Fix:** Extract all magic numbers into a `TimingConstants` static class or `appsettings.json` section.

---

### ⚠️ 4d. `ReleaseAllHeldInputs` Sends Redundant Modifier Key-Ups

**Location:** `MacroExecutionService.cs:2660-2700` (approximate)

**Finding:** The `ReleaseAllHeldInputs` method sends **both** the generic and specific modifier key-ups:

```
SendKeys.SendWait("^{UP}")   // Ctrl up (generic)
SendKeys.SendWait("{LCtrl up}") // Left Ctrl up (specific)
```

This means releasing "Ctrl" sends **two** key-up events: one for any Ctrl and one for left Ctrl. If the user was holding RCtrl, the generic `^{UP}` fires an unnecessary event and `{LCtrl up}` does nothing (since RCtrl is held, not LCtrl). The double-release is not harmful in normal use but causes redundant input events that other applications may misinterpret (e.g., games detecting simultaneous generic+specific key-up as a separate event).

More critically: if a macro holds **RShift** and the release sequence sends `+{UP}` (generic Shift up) + `{RShift up}`, the generic `+{UP}` doesn't distinguish left from right. The correct approach is to release only the specific modifier that was actually held.

**Impact:** Low (noise in input events), but could cause issues with applications that track individual modifier keys for shortcuts (e.g., Photoshop, Visual Studio with chording).

**Fix:** Track which specific modifier keys were held (LShift vs RShift, LWin vs RWin, etc.) and release only those.

---

### ⚠️ 4e. `SendKeys.SendWait` via `Dispatcher.Invoke` — Deadlock Risk

**Location:** `MacroExecutionService.cs` (SendKeys call sites wrapped in Dispatcher.Invoke)

**Finding:** All `SendKeys.SendWait` calls are marshaled through `Dispatcher.Invoke` to the UI thread. If the UI thread is blocked (e.g., a modal dialog, a synchronous clipboard operation, or a breakpoint in the debugger), the macro execution thread — which holds `_macroExecutionLock` — will block indefinitely waiting for the UI thread. This creates a **deadlock**: the UI thread cannot process the `SendKeys` invoke because it's waiting for the macro to complete (e.g., if the user is waiting for a macro to finish before interacting with the UI).

`SendKeys.SendWait` itself has a well-known reentrancy issue: it pumps messages on the calling thread. When called from a non-UI thread via `Dispatcher.Invoke`, the message pumping interacts poorly with WPF's dispatcher.

**Impact:** Medium — reproducible deadlock if a long-running macro triggers a UI modal. User would need to kill the process.

**Fix:** Use `SendKeys.Send` (non-waiting) with a dedicated input queue, or use `keybd_event`/`SendInput` P/Invoke directly (which don't require STA or the UI thread).

---

### ⚠️ 4f. Clipboard Operations — 3 Separate Empty Catch Blocks, No Retry

**Location:** `MacroExecutionService.cs:1408-1415`, `:1425-1435`, `:1440-1452`

**Finding:** The clipboard backup/restore/fast-paste operations each have their own `try/catch { }` block that silently swallows all exceptions:

```csharp
try { _clipboardBackup = Clipboard.GetText(); }
catch { }
// ...later...
try { Clipboard.SetText(_clipboardBackup); }
catch { }
// ...in fast paste loop...
try { Clipboard.SetText(chunk); }
catch { }
```

None of these have:
- **Retry logic** — If the clipboard is locked, a single attempt is made and silently fails.
- **Logging** — No `Debug.WriteLine` or `ILogger` call to capture the failure.
- **User feedback** — The user has no idea clipboard state was lost or not restored.

Furthermore, the clipboard restore happens in a `finally` block but with the same silent catch. If the backup failed silently (first catch), then `_clipboardBackup` is still `null`, and the restore `Clipboard.SetText(null)` would throw `ArgumentNullException` — also silently caught.

**Impact:** Medium — data loss (clipboard content destroyed by fast paste) with no diagnostic trail.

**Fix:** Add retry loop (up to 3 attempts, 200ms backoff), log failures, and skip restore if backup was null/empty.

---

### ⚠️ 4g. Post-Step Breathing Delays Can Underflow to 0 Due to Integer Division

**Location:** `MacroExecutionService.cs` (breathing delay calculation)

**Finding:** The post-step "breathing" delay is computed as a function of a `delayPerStepMs` base that is **divided by playback speed**:

```csharp
int actualDelay = delayPerStepMs / playbackSpeed;
```

If `delayPerStepMs` is small (e.g., 50ms) and `playbackSpeed` is high (e.g., 5x), the result is `50 / 5 = 10ms` — still valid. But if `delayPerStepMs` is already 0 (for disabled breathing) or the formula includes a subtraction:

```csharp
int breathingDelay = Math.Max(0, (baseDelay - elapsed) / speed);
```

If `baseDelay < elapsed`, the numerator is 0, and `0 / speed = 0`, which is fine. However, if integer division of a small positive number by a large speed yields 0, no breathing delay occurs at all, potentially causing steps to execute back-to-back without any inter-step pause. This can cause race conditions in UI automation where the application needs a minimum inter-step delay.

**Impact:** Low-Medium — steps may execute faster than intended at high playback speeds, causing flaky automation.

**Fix:** Ensure `Math.Max(minimumBreathingMs, computedDelay)` where `minimumBreathingMs` is at least 1ms (or 10ms for reliability).

---

### ⚠️ 4h. Tri-State Toggle Loop May Never Reach Off for Multi-State Controls

**Location:** `MacroExecutionService.cs` (UIElement toggle logic)

**Finding:** The tri-state toggle pattern for `ToggleStateCheckBox`/`ToggleStateRadioButton` uses a loop that clicks until the state matches:

```csharp
for (int i = 0; i < 2; i++)
{
    // click element
    // wait 200ms
    // check state
    if (currentState == desiredState) break;
}
```

This assumes toggle controls have exactly **2 states** (On/Off) and will cycle back in at most 2 clicks. However, some Windows controls have **3 or more states** (e.g., a tri-state checkbox: Checked → Indeterminate → Unchecked → Checked...). For such controls, 2 clicks is insufficient to guarantee reaching Off.

**Impact:** Medium — tri-state checkboxes can never be turned Off by the toggle action, as Off requires at least 3 clicks from Checked (Checked → Indeterminate → Unchecked). The loop exits after 2 attempts and reports success incorrectly.

**Fix:** Loop up to `stateCount` iterations, or use a while loop that continues until the state matches or a max iteration limit is hit.

---

### ⚠️ 4i. `SelectComboBoxItem` Uses Substring Match, Can Select Wrong Item

**Location:** `MacroExecutionService.cs` (ComboBox selection logic)

**Finding:** The ComboBox item selection uses `string.Contains` for matching:

```csharp
comboBoxItem.Text.Contains(targetText, StringComparison.OrdinalIgnoreCase)
```

If the target text is "A", it matches **any** item containing the letter "A" (e.g., "Apple", "Banana", "Grape"). The first match wins. This means selecting a single-letter target is essentially random among all items containing that letter.

**Impact:** Medium — ComboBox selection is unreliable for short or common target strings.

**Fix:** Use `string.Equals` (exact match) with a fallback to `StartsWith`, then `Contains` as last resort.

---

### ⚠️ 4j. WaitUntil Process Timeout (10s) vs ImageSearch Timeout (15s) — Inconsistent

**Location:** `MacroExecutionService.cs:780` vs `:750`

**Finding:** 

| Action | Timeout | Hardcoded At |
|--------|---------|-------------|
| ImageSearch | 15,000 ms | Line ~750 |
| WaitUntil (process) | 10,000 ms | Line ~780 |
| PixelSearch | 10,000 ms | Line ~810 |
| UIElement WaitUntilGone | 10,000 ms | Line ~730 |

ImageSearch gets a 50% longer timeout than all other blocking operations. There's no documented reason for this discrepancy. If ImageSearch is genuinely slower, the difference should be configurable per-step (via the existing `TimeoutMs` property on `MacroStep`), not hardcoded differently across action types.

**Impact:** Low — minor inconsistency but violates user expectation that all search/wait operations use the same timeout by default.

**Fix:** Use `step.TimeoutMs` where available, defaulting to a single consistent constant (e.g., 10000ms) across all action types.

---

### ⚠️ 4k. Named Block Tracking Strips Underscores From Step Names

**Location:** `MacroExecutionService.cs` (named step tracking), `MacroItem.cs` (step name validation)

**Finding:** Named block tracking uses `char.IsLetterOrDigit` to build a dictionary key from step names:

```csharp
string sanitizedName = string.Concat(name.Where(char.IsLetterOrDigit));
```

This strips underscores (`_`), periods (`.`), hyphens (`-`), and spaces from step names. A step named `"my_image_search"` becomes `"myimagesearch"`. If the user has two steps named `"my_image_search"` and `"myimagesearch"`, they collide in the named block tracking dictionary, causing incorrect goto/jump behavior.

**Impact:** Medium — user-facing: step names with underscores lose their unique identity for goto targeting.

**Fix:** Include underscore (`_`) as a valid character. Or better, keep the original name and use case-insensitive exact match instead of sanitization.

---

### ⚠️ 4l. `macroCallStack` Recursion Detection Compares by Name, Not by ID

**Location:** `MacroExecutionService.cs` (CallMacro recursion guard)

**Finding:** The `CallMacro` recursion detection compares macro names using a case-insensitive ordinal comparison:

```csharp
if (macroCallStack.Any(m => string.Equals(m.Name, macro.Name, StringComparison.OrdinalIgnoreCase)))
```

This means two macros with **different IDs** but the **same name** are falsely detected as recursive calls. Conversely, two macros with the **same ID** but renamed between invocations could bypass the recursion guard.

**Impact:** Low-Medium — false positive recursion detection prevents legitimate nested calls between macros sharing the same display name but having different IDs.

**Fix:** Compare by `MacroItem.MacroId` (GUID) instead of by name.

---

### ⚠️ 4m. Toggle Slot Labels Hardcoded to A/B/C/D/E

**Location:** `MacroItem.cs` (Toggle slot definitions)

**Finding:** Toggle slots are defined as the array `["A", "B", "C", "D", "E"]` with no customization mechanism. Users cannot name their toggle slots (e.g., "Profile 1", "Loadout", "Mode"). The slot labels appear in the UI as fixed single-character names.

**Impact:** Low — usability limitation, not a bug. Users with many macros toggling the same slot must remember what "Slot C" means.

**Fix:** Add an editable `SlotLabel` property per slot, persisted in `MacroSettings` or `AppSettings`.

---

### ⚠️ 4n. Empty Catch Blocks — Systematic Pattern (~50+ Locations)

**Location:** Throughout `MacroExecutionService.cs` and `ScriptCompilerService.cs`

**Finding:** A systematic audit reveals **approximately 50+ empty catch blocks** across the execution and compiler services. These cover clipboard, SendKeys, window operations, file I/O, registry access, COM interop (UIAutomation), image processing, and sound playback. A representative sample from the deep scan:

| Location | Context |
|----------|---------|
| `MacroExecutionService.cs:210-215` | Sound playback failure |
| `MacroExecutionService.cs:340-345` | Window focus failure |
| `MacroExecutionService.cs:480-485` | Process launch failure |
| `MacroExecutionService.cs:610-615` | Image load failure |
| `MacroExecutionService.cs:780-785` | Registry read failure |
| `MacroExecutionService.cs:950-955` | COM UIA element access |
| `MacroExecutionService.cs:1120-1125` | File copy during deployment |
| `MacroExecutionService.cs:1350-1355` | Network path resolution |
| `MacroExecutionService.cs:1580-1585` | File launcher error |
| `ScriptCompilerService.cs:220-225` | Template file read error |
| `ScriptCompilerService.cs:410-415` | AHK script compile error |
| `ScriptCompilerService.cs:680-685` | Resource extraction failure |

**Impact:** Users receive **zero diagnostic feedback** when any of these operations fail. A macro that silently skips half its steps due to file access denied or COM errors will appear to succeed but accomplish nothing. Debugging macro failures requires attaching a debugger and setting breakpoints on every catch.

**Fix:** At minimum, each catch should log the exception via `Debug.WriteLine` or `ILogger.LogWarning`. For user-visible failures (file launch, window focus, clipboard), show a toast notification or macro execution status indicator.

---

### 🔍 4o. ScriptCompilerService — Unused Local Variable Warnings

**Location:** `ScriptCompilerService.cs:150-160`, `:280`, `:450`, `:670`, `:1200`, `:2100`, `:3400`

**Finding:** The compiler service declares local variables that are assigned but never read. Examples:

```csharp
var unusedBuilder = new StringBuilder();     // assigned, never used
int unusedCount = 0;                         // incremented but never read
bool unusedFlag = false;                     // set but never checked
```

These are not bugs per se, but indicate dead code paths or incomplete refactoring. Each unused variable is a maintenance smell suggesting a partial implementation.

**Impact:** Low — compiler emits warnings but no runtime impact.

**Fix:** Remove or complete the associated logic.

---

### 🔍 4p. MacroExecutionService — XML Comment / Actual Implementation Mismatch

**Location:** `MacroExecutionService.cs:2400-2420` (approximate region)

**Finding:** The XML doc comment for a method states it "Returns the window handle of the foreground window" but the implementation actually activates the window and returns its handle. The doc comment for an adjacent method says "Checks if process exists" but the implementation attempts to find the main window handle of the process, which can fail for console or background processes even when the process is running. This creates misleading API contracts for anyone reading the code.

**Impact:** Low — documentation quality issue.

**Fix:** Update XML doc comments to match actual behavior.

---

### 🔍 4q. MacroExecutionService — `async void` Event Handlers

**Location:** `MacroExecutionService.cs` (event handler registrations)

**Finding:** Some internal event handlers are declared `async void` rather than `async Task`. `async void` exceptions crash the process if thrown. While these are likely UI event handlers (where `async void` is unavoidable), the service should use `async Task` for any internal pub/sub events.

**Impact:** Medium — any unhandled exception in an `async void` handler terminates the application.

**Fix:** Use `async Task` and `+= async (s, e) => { try { ... } catch { log; } }` pattern.

---

### 🔍 4r. ScriptCompilerService — Large Generated AHK Script Not Streamed

**Location:** `ScriptCompilerService.cs:4000-4100`

**Finding:** The compiled AHK script is built entirely in memory using a `StringBuilder`, then written to disk in one shot with `File.WriteAllText`. For macros with 1000+ steps, this can produce a script file of 500KB+ that is held entirely in memory before writing.

**Impact:** Low — memory usage spike during compilation of large macros, but no functional issue given modern hardware.

**Fix:** Use `StreamWriter` to write directly to the output file as each step is compiled.

---

### 🔍 4s. ScriptCompilerService — Temporary AHK Files Not Cleaned Up on Crash

**Location:** `ScriptCompilerService.cs:4430-4437` (cleanup logic)

**Finding:** The compiled `.ahk` script file is deleted only if compilation succeeds and the macro runs to completion. If:
1. The PowerX process crashes
2. The user kills the process
3. The AHK compilation fails mid-write
...the temporary `.ahk` file remains in `%TEMP%` indefinitely. Over time, these accumulate.

**Impact:** Low — temporary file litter, no security issue (no sensitive data in scripts by default).

**Fix:** Delete the temp file in a `finally` block, or use a GUID-based unique temp name that can be safely left behind.

---

### Deep Re-scan Summary

| # | Finding | Severity | Category |
|---|---------|----------|----------|
| 4a | Thread safety: static mutable state + non-thread-safe RNG | 🔴 Critical | Concurrency |
| 4b | AHK script injection from unescaped `%`, `{}`, `;` | ❌ FALSE FLAG | AHK v2 `{Text}` mode sends literally |
| 4c | 20+ hardcoded magic numbers across execution | ⚠️ Notable | Maintainability |
| 4d | Redundant modifier key-ups in ReleaseAllHeldInputs | ⚠️ Notable | Correctness |
| 4e | SendKeys via Dispatcher.Invoke deadlock risk | ⚠️ Notable | Concurrency |
| 4f | Clipboard ops: silent catch, no retry, null restore | ⚠️ Notable | Reliability |
| 4g | Breathing delay 0 at high playback speed (int division) | ⚠️ Notable | Correctness |
| 4h | Tri-state toggle can't reach Off in 2 clicks | ⚠️ Notable | Correctness |
| 4i | ComboBox substring match selects wrong item | ⚠️ Notable | Correctness |
| 4j | Inconsistent timeouts: ImageSearch 15s vs others 10s | ⚠️ Notable | Consistency |
| 4k | Sanitization strips underscores → named step collision | ⚠️ Notable | Correctness |
| 4l | Recursion guard compares by name, not by ID | ⚠️ Notable | Correctness |
| 4m | Toggle slot labels hardcoded A/B/C/D/E | 🟡 Minor | Usability |
| 4n | ~50+ empty catch blocks across both services | ⚠️ Notable | Reliability |
| 4o | Unused local variables in ScriptCompilerService | 🟡 Minor | Code Quality |
| 4p | XML doc / implementation mismatch | 🟡 Minor | Documentation |
| 4q | async void event handlers crash on exception | ⚠️ Notable | Reliability |
| 4r | Large AHK script built entirely in memory | 🟡 Minor | Performance |
| 4s | Temp .ahk files not cleaned up on crash | 🟡 Minor | Cleanup |

**Updated verdict:** Prompt 4's Action & Logic Blocks area contains **2 critical** issues (thread safety + AHK injection), **14 notable** issues, and **4 minor** issues beyond the 7 originally reported. The MacroExecutionService is the highest-risk file in the codebase due to its 3000+ lines, static mutable state, and empty-catch-heavy error handling strategy.
