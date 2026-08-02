# 🧠 Brainstorming Request: Window Block Architecture & Improvements

We are seeking creative ideas to improve the **Window Block (WindowAction step)** in PowerX Keys — a Windows desktop macro automation app built with C# .NET 10, WPF, AutoHotkey v2, and SQLite.

---

## 🎯 What Is The Window Block?

The **Window Block** is a macro step that performs an action on a specific application window. It is one of the most fundamental building blocks in PowerX Keys — almost every macro starts with a Window Block to bring the right app to focus before interacting with it.

### Supported Actions:
| Action | What It Does |
|---|---|
| **Activate** | Bring the target window to foreground and focus it (most common) |
| **Close** | Close the target window |
| **Minimize** | Minimize the target window |
| **Maximize** | Maximize the target window |
| **Move** | Reposition and/or resize the window to recorded geometry |

---

## ⚙️ How The Activate Action Works (Most Complex)

The Activate action is far more sophisticated than a simple `WinActivate()`. It compiles into a **multi-step smart cascade** in AHK v2 via `ScriptCompilerService.cs`:

### Step 1: Check If Target Is Already Active
```ahk
if (_ActiveHwnd && WinActive("My Window Title ahk_exe chrome.exe")) {
    _TargetHwnd := _ActiveHwnd
    LastActionSucceeded := 1
}
```
If the exact target is already in focus — done instantly. Zero overhead.

### Step 2: Browser Tab Switch (Lightning Fast)
If the target is a browser tab (Chrome, Edge, Brave, Firefox) and the browser is already open:
```ahk
if (!_TargetHwnd && _ActiveHwnd && WinActive("ahk_exe chrome.exe")) {
    ; UIAutomation COM scan of tab bar
    ; Find tab whose title matches "My Window Title"
    ; Call SelectionItem.Select() to switch to it
    ; Fallback: Click the tab's center coordinates via UIA bounding rect
}
```
This switches browser tabs **without leaving the browser window** — purely via UIAutomation, no keyboard shortcuts.

### Step 3: Search Other Windows (Optional)
If "Search Other Windows" checkbox is enabled:
```ahk
; Find another window matching exact title
_OtherHwnd := WinExist("My Window Title ahk_exe chrome.exe")
if (_OtherHwnd) {
    ; Smart restore: if minimized → WinRestore → Sleep(50)
    ; WinShow → WinSetAlwaysOnTop(1) → WinActivate → WinSetAlwaysOnTop(0)
    ; Optional: WinMove to recorded size/position
}

; Try tab switch on all OTHER open browser windows
; Then: Ultimate fallback — activate ANY window of this exe
_AnyHwnd := WinExist("ahk_exe chrome.exe")
```

### Step 4: Auto-Launch If Missing (Optional)
If the window doesn't exist anywhere and "Auto-Launch" is enabled:
```ahk
ToolTip("Target missing. Attempting to launch...")
try {
    Run("C:\path\to\app.exe")
    _TargetHwnd := WinWait("My Window Title", , 10)
} catch {
    try { Run("C:\path\to\app.exe") } catch {}  ; Fallback raw launch
    _TargetHwnd := WinWait("My Window Title", , 5)
}
```

### Special Case: Desktop Window
The Windows desktop (`explorer.exe / Progman`) cannot be activated with `WinActivate`. Special path:
```ahk
; Smart Desktop Click — minimize all windows to reveal desktop
try {
    ComObject("Shell.Application").MinimizeAll()
} catch {
    Send("#d")  ; Fallback: Win+D
}
Sleep(400)
```

---

## 🔧 Key MacroStep Properties for Window Block

| Property | Purpose |
|---|---|
| `WindowTitle` | The recorded window title string (e.g. `"My Doc - Notepad"`) |
| `Value` | The action: `Activate`, `Close`, `Minimize`, `Maximize`, `Move` |
| `SearchOtherWindows` | Whether to search non-active windows if target not found |
| `AutoLaunchIfMissing` | Auto-launch the app if window doesn't exist |
| `AutoLaunchPath` | Custom exe path to launch (fallback uses WindowTitle) |
| `SmartWait` | Insert `WinWait(title, , 10)` before activating (for slow-loading apps) |
| `CaptureWindowParameters` | Whether to also restore the window to recorded size/position |
| `WindowX`, `WindowY`, `WindowWidth`, `WindowHeight` | Recorded window geometry |
| `BrowserTabSwitchEnabled` | Enable smart browser tab switching via UIAutomation |
| `BrowserTabName` | The browser tab title to switch to |
| `IsDesktopWindow` | Marks this as a desktop target (uses MinimizeAll instead of WinActivate) |
| `FailIfMissing` | Abort macro if window cannot be found or activated |

---

## 🐞 Current Pain Points & Known Limitations

1. **Window Title Brittleness**: AHK matches window titles by substring (SetTitleMatchMode 2). If a document name changes (e.g. "Report.docx - Word" vs "Report_v2.docx - Word"), the block fails because the title no longer matches.

2. **No Process-Only Matching Fallback**: If an exact title match fails and `SearchOtherWindows` is off, there's no graceful "find any window of this exe" fallback unless the user explicitly enables it.

3. **Multi-Instance Apps**: If the user has 2 Chrome windows open, the cascade picks the first one it finds via `WinExist`. There's no way to specify "the second Chrome window" or "the Chrome window that was previously focused".

4. **Slow Apps**: `SmartWait` uses `WinWait` which blocks. If an app never opens, the macro hangs for 10 seconds before aborting. There's no progress indicator or timeout config per-step.

5. **Window Position Restore Is Rough**: `WinMove` happens immediately after activation with no check of whether the window is ready. On slow machines or high-DPI setups, the window may not have fully rendered before being moved.

6. **Browser Tab Matching Is Fragile**: Tab title matching strips "- Google Chrome" / "- Microsoft Edge" etc., then does a case-insensitive substring match. But tab titles can be dynamic (e.g. count badges: "Gmail (3)"), causing mismatches. The regex `\(\d+\)` is stripped but other dynamic suffixes are not handled.

7. **Auto-Launch Has No Startup Delay Config**: After `Run()`, the macro waits up to 10 seconds via `WinWait`. But heavy apps (e.g. Photoshop) may take 30+ seconds. There's no user-configurable timeout per step.

8. **No "Wait Until Window Closes" Action**: There's `WinWaitClose()` used internally but it's not exposed as a user-selectable action in the Window Block.

9. **No Window State Check Before Activate**: The smart activate always does `WinSetAlwaysOnTop(1)` then `WinActivate` then `WinSetAlwaysOnTop(0)`. This flickers the taskbar and may cause visual issues on some systems.

10. **Desktop Detection Is Heuristic**: `IsDesktopWindow` is set during recording but if someone edits the step manually or it wasn't detected correctly, the macro silently uses wrong behavior.

---

## 💡 Seed Ideas (Not Finalized)

- **Fuzzy Title Matching**: Instead of exact substring, allow configurable fuzzy matching (e.g. match any window whose title is 80% similar to the recorded one). Handles document name changes.
- **Window Identity by Process + Position**: Identify the target window not just by title, but by a combination of exe name + approximate screen position at record time. More robust for renamed documents.
- **Configurable WinWait Timeout**: Expose `SmartWait` timeout as a per-step setting (default 10s, max 60s).
- **"Ensure Window Size" Mode**: A dedicated step option that checks if window is the right size and only moves it if it differs — avoids jarring reposition on windows that are already correct.
- **Wait Until Window Closes Action**: Expose `WinWaitClose` as a proper action option.

---

## 🚀 Your Instructions

Please evaluate the current Window Block system and provide:
1. **Assessment of the current multi-step cascade**: What is architecturally strong vs. brittle?
2. **Ideas to make window targeting more reliable**: Especially for renamed documents, multi-instance apps, and browser tabs.
3. **Ideas to improve UX**: How should users configure and understand window matching?
4. **Any radical rethinks**: Is there a better way to identify and activate a target window that is more resilient than title-based matching?

Write your feedback and ideas directly into this file under your agent section!

---

## ✏️ Agent Responses

### Agent 1 Feedback & Ideas:

**1. Architecture Assessment**

Strong: The multi-step cascade (active check → tab switch → search other windows → auto-launch) is well-thought-out and handles the vast majority of real-world scenarios. The browser tab switching via UIA is genuinely innovative. The Desktop special-case (MinimizeAll) is correct — WinActivate on Progman is impossible.

Brittle: Window title substring matching is the weakest link. If the title changes even slightly, the cascade falls to Step 4 (auto-launch) which tries to re-launch an already-running app. That's both wrong (the app is running, just not found) and slow (10s WinWait timeout). The cascade should have a "match any window of this EXE" fallback between Step 3 and Step 4 as a built-in default, not an optional checkbox.

**2. Ideas for More Reliable Window Targeting**

- **Exe-Only Fallback (Built-in)**: Between "search other windows" and "auto-launch", add an implicit step: find ANY window belonging to the target process, regardless of title. If the user recorded `chrome.exe` with title "Gmail", and the title changed to "Inbox (8) - Gmail", the exe-only match catches it. This should be on by default, not hidden behind "Search Other Windows".

- **Window Fingerprinting**: At record time, capture a fingerprint of the window: process name + approximate position + size + a hash of its visible UI element tree (top 3 UIA elements). At match time, use title as primary hint but verify by fingerprint. Handles renamed documents perfectly.

- **MRU Window Ordering**: For multi-instance apps, instead of picking the first window WinExist returns, order candidates by most-recently-used (via `WinGetList` + `WinActive` check history). Activate the one that was most recently interacted with.

**3. UX Improvements**

- Show the match confidence in the macro editor (e.g. "Matched by title ✓", "Matched by exe only ⚠", "Not found ✗") so users can see if their window target is getting brittle before the macro fails.

- Add a "Test Target" button that runs the cascade in preview mode (no actual activation) and reports which step hit + the matched window title, so the user can verify targeting without disrupting their desktop.

**4. Radical Rethink: Window ID by Process + Creation Time**

Instead of matching windows dynamically at runtime, identify the window by its process ID + start time at the moment the macro runs. This naturally handles renamed documents because you're not matching by title at all — you're matching "the Notepad window that was opened 30 seconds ago". This works brilliantly for macros that launch an app and immediately interact with it, which is the most common pattern.

For already-running apps, fall back to the fingerprinting approach described above.

---

### Agent 2 Feedback & Ideas:
*(Write your thoughts and suggestions here)*

---

### Agent 3 Feedback & Ideas:
*(Write your thoughts and suggestions here)*

You can paste this directly under **Agent 2**.

---

## **1. Architecture Assessment**

The current activation cascade is excellent because it follows the natural recovery path:

```
Already Active
↓

Browser Tab

↓

Other Windows

↓

Launch Application
```

However, every decision is based on **how to find the window**, not **how confidently we've identified it**.

I would separate the architecture into two independent systems:

* **Window Discovery**
* **Window Verification**

Discover many candidates first.

Then verify which one is actually the intended target.

This makes the system much more resilient for multi-instance applications.

---

## **2. Window Identity Profile**

Instead of storing only:

```
Title

Exe
```

Record an entire identity profile.

Example:

```
Executable

Window Class

Process Name

Process Path

Monitor

Approx Position

Approx Size

Parent Process

UIA Root Name

Window Style

Window Icon Hash
```

None of these alone are reliable.

Together they form a very strong fingerprint.

Runtime simply scores every candidate.

Highest score wins.

---

## **3. Confidence-Based Matching**

Instead of

Found / Not Found

calculate:

```
Window A

94%

Window B

73%

Window C

21%
```

Example scoring:

* Title match
* EXE match
* Window class
* Position similarity
* Size similarity
* UIA fingerprint
* Last activated history

This avoids activating the wrong Chrome window.

---

## **4. Window Fingerprint Cache**

Whenever a Window Block succeeds,

save:

```
HWND

PID

Creation Time

Position

Size

```

If the same macro runs again shortly afterward,

first check whether that exact window still exists.

If yes,

activation becomes nearly instantaneous.

---

## **5. Smart Title Rules**

Instead of raw substring matching,

normalize titles before comparison.

Automatically remove:

```
(12)

•

*

[Administrator]

- Google Chrome

- Microsoft Edge

— Mozilla Firefox
```

Also allow user-defined ignore patterns.

Example:

```
Ignore

*

(.* unread.*)

Draft

Loading...
```

This dramatically improves browser and document matching.

---

## **6. Window Intent**

Instead of asking

"Which title?"

let the user specify intent.

Examples:

```
Current Chrome

Last Active Chrome

Newest Chrome

Oldest Chrome

Largest Chrome

Chrome On Monitor 2

Any Chrome
```

This solves many multi-instance scenarios without exposing technical complexity.

---

## **7. Visual Verification**

Before activation,

capture a tiny screenshot of the title bar during recording.

At runtime,

after a candidate window is found,

compare only the title bar region.

This confirms

Yes

this is probably the same application window.

Very lightweight.

Far cheaper than full ImageSearch.

---

## **8. Activation Report**

After every Window Block:

```
Matched

Chrome.exe

Method

UIA Tab

Confidence

97%

Time

14 ms
```

If confidence is low:

```
⚠

Matched by EXE only
```

Extremely useful for debugging large macros.

---

## **9. Learning Window Preferences**

If users repeatedly select the second Chrome window,

PowerX Keys remembers.

Eventually:

```
Preferred Chrome Window

Window #2

Confidence

High
```

The software gradually learns the user's workflow instead of always selecting the first match.

---

## **10. Window Studio**

Create a dedicated Window Inspector.

Displays:

```
HWND

PID

Class

Styles

Monitor

DPI

Title

Owner

Children

UIA Tree

Browser Tabs

Executable
```

Users can immediately understand why a window isn't matching.

An invaluable debugging tool.

---

## **11. Preview Match Simulator**

Inside the editor:

```
Candidate 1

95%

✓ Selected

Candidate 2

82%

Candidate 3

37%
```

Before saving,

users can see exactly which window would be activated.

No surprises during runtime.

---

## **12. Window Relationship Matching**

Sometimes users don't actually care about the window.

They care about its relationship.

Examples:

```
Window

owned by

Explorer

```

or

```
Child

of

Visual Studio
```

or

```
Same process

as previous step
```

Relationship-based matching is often more stable than title matching.

---

## **13. Macro Session Memory**

Allow Window Blocks to remember discovered windows.

Example:

```
Window Block

↓

Launch App

↓

PID = 8144

↓

Activate

↓

Resize

↓

Close
```

Subsequent Window Blocks don't need to rediscover the window—they simply reuse the cached handle (after validating it's still alive).

This makes long automation sequences significantly faster and more reliable.

---

## **14. Radical Rethink: Window Resolver Engine**

Rather than making Window Block itself responsible for every matching strategy, introduce a centralized **Window Resolver Engine**.

Every Window Block simply asks:

> "Resolve my target window."

The resolver internally combines:

* HWND cache
* Process matching
* Title matching
* UIAutomation fingerprint
* Position similarity
* Window class
* Previous macro session memory
* Confidence scoring

It returns the best candidate plus a confidence value.

This centralizes all window-identification logic in one place, making every Window Block automatically benefit from future improvements without changing the block itself.


### Agent 2 Feedback & Ideas:

**1. Architecture Assessment**

**Strong**
- The cascade order is excellent: cheapest check first (already active), then specialized fast path (browser tab UIA), then broader search, then last-resort launch. This is the correct performance hierarchy.
- Browser tab switching via UIAutomation without leaving the browser is a genuine competitive advantage — most macro tools still spam Ctrl+Tab or click coordinates.
- Desktop special-case handling is correctly isolated.
- The “already active → instant success” short-circuit is the right optimization for the most common case.

**Brittle**
- Title matching is the single point of failure that poisons the entire cascade. Once the title drifts, the block either fails or falls into the expensive auto-launch path even though the app is already running.
- “Search Other Windows” being optional means many users never enable the safety net they actually need.
- Multi-instance handling is essentially non-existent — `WinExist` just returns the first match, which is non-deterministic.
- Geometry restore (`WinMove`) is fire-and-forget with no readiness check, leading to races on slow or high-DPI machines.
- No intermediate “any window of this process” tier means the cascade jumps too quickly from “exact title” to “launch a new instance”.

**2. Ideas to Make Window Targeting More Reliable**

- **Process-First Matching with Title as Hint**  
  Change the default identity model: primary key = executable name (+ optional class name). Title becomes a soft preference, not a hard requirement.  
  Cascade becomes:  
  1. Exact title match (fast path)  
  2. Same process + closest title similarity (fuzzy)  
  3. Same process + most recently active  
  4. Same process + any window  
  5. Auto-launch  
  This single change eliminates the majority of “document renamed” failures.

- **Fuzzy Title Scoring**  
  When exact match fails, compute a simple similarity score (Levenshtein or token overlap after stripping common suffixes like “- Google Chrome”, “(3)”, “.docx”). Prefer the highest-scoring window of the correct process. Configurable threshold (default 70 %).

- **Instance Disambiguation**  
  For multi-instance apps, store three pieces of context at record time:  
  - Process name  
  - Approximate screen position / size  
  - “Was this the foreground window when recorded?”  
  At runtime rank candidates by: recently active → position proximity → size proximity. Users can also explicitly choose “Always the most recent”, “Always the leftmost”, or “Always the one at recorded position”.

- **Stable Window Class + Process Pair**  
  Many apps have stable `ahk_class` values even when titles change. Prefer matching `ahk_exe + ahk_class` first, then use title only for disambiguation among multiple windows of the same class.

- **Browser Tab Resilience**  
  In addition to stripping count badges, maintain a short list of known dynamic suffixes (notification counts, “ - Unsaved”, etc.) and also try matching the *base* title tokens. Optionally fall back to matching the tab’s favicon or a UIA AutomationId if available.

- **Geometry Restore with Readiness Guard**  
  After `WinActivate`, wait until the window reports a non-zero client area *and* is not minimized, then apply `WinMove`. Add a short configurable settle delay (default 80–120 ms) that can be overridden per step.

**3. UX Improvements**

- **Match Strategy Preview Chip**  
  In the step editor, show a live chip:  
  `Title Exact` / `Fuzzy Title` / `Process Only` / `Most Recent Instance`  
  Users immediately understand how brittle (or robust) their current configuration is.

- **“Test Cascade” Button**  
  Runs the full matching logic in dry-run mode and reports:  
  - Which tier succeeded  
  - The actual window title + HWND that would have been activated  
  - How long each tier took  
  No desktop disruption, instant feedback.

- **Smart Defaults on Capture**  
  When the user records a Window Block:  
  - Detect if the title looks document-like (contains file extension or “ - AppName”) → default to Process-First mode.  
  - Detect browser → enable tab switching + store cleaned tab name.  
  - Detect multiple instances of the same exe → prompt “Which instance should this always target?”

- **Timeout & Retry Controls Exposed**  
  Make `SmartWait` timeout a visible per-step slider (1–60 s). Also expose a “Retry count” for activation attempts with short delays between them.

- **Visual Identity Card**  
  In the properties panel show a small card: process icon + cleaned title + “Last matched via: Fuzzy (82 %)” so the user can see at a glance whether the target is still healthy.

**4. Radical Rethinks**

- **Window Handle Lifetime Tracking**  
  When a macro launches an app or successfully activates a window, store the HWND (or a short-lived PID + creation timestamp) in a macro-scoped variable. Subsequent Window Blocks in the same macro can prefer that exact handle. This is extremely reliable for the common “launch → interact → launch another → interact” pattern and completely sidesteps title changes.

- **UIA Root Anchor (Optional)**  
  For apps that expose a stable AutomationId or Name on their main window, allow the user to optionally bind to that UIA element instead of (or in addition to) the classic WinTitle. Activation then becomes “find the UIA element → bring its window to front”. Survives title changes and works well for modern UWP / WinUI apps.

- **Soft Activate vs Hard Activate**  
  Offer two modes:  
  - Soft: only activate if the window is not already the foreground window of its process (avoids unnecessary AlwaysOnTop flicker).  
  - Hard: force activation + optional geometry restore (current behavior).  
  Most macros only need Soft.

**Highest-ROI Immediate Actions**
1. Make “any window of this process” a built-in tier (not optional).  
2. Switch default matching to Process-First + fuzzy title.  
3. Add a dry-run “Test Cascade” button.  
4. Guard geometry restore with a readiness check + short settle delay.

These four changes turn the Window Block from “works until the title changes” into “works across document renames, multi-instance, and slow apps” while preserving the excellent cascade structure already in place.