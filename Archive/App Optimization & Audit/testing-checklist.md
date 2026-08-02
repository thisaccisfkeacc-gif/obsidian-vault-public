---
tags: [status, testing, qa, checklist]
date: 2026-06-10
status: in-progress
---


# PowerX Keys — Master QA Checklist 🧪

> **Instructions:** Test each item and update status:
> - `[ ]` = Not tested yet
> - `[x]` = ✅ Working correctly
> - `[!]` = 🐛 Bug found (add details in Notes)
> - `[-]` = ⏭️ Skipped / Not applicable

---

## Phase 1: App Startup & General 🚀

| #   | Test                                         | Status | Notes                                                   |
| --- | -------------------------------------------- | ------ | ------------------------------------------------------- |
| 1   | App launches with no errors or crashes       | `[x]`  | ✅ Working                                               |
| 2   | Minimize to tray — app hides to system tray  | `[x]`  | ✅ Working                                               |
| 3   | Click tray icon — app restores from tray     | `[x]`  | ✅ Working                                               |
| 4   | Right-click tray icon — context menu appears | `[x]`  | ✅ Working                                               |
| 5   | Exit from tray — app closes completely       | `[x]`  | ✅ Working                                               |
| 6   | App version badge visible in header          | `[x]`  | ✅ By design — badge is in Settings / What's New section |
| 375 | Splash screen with fade-out animation        | `[ ]`  |                                                         |
| 376 | Single instance — second launch restores first | `[ ]`  |                                                       |
| 377 | Crash report dialog on next launch           | `[ ]`  |                                                         |
| 378 | Engine auto-restart after PC sleep/wake      | `[ ]`  |                                                         |
| 379 | Auth window blocks startup until sign-in     | `[ ]`  |                                                         |
| 380 | Subscription expired window blocks app       | `[ ]`  |                                                         |
| 381 | Force update blocks app when flagged         | `[ ]`  |                                                         |
| 382 | Auto-update check on startup (background)    | `[ ]`  |                                                         |
| 383 | Scheduled macro via --run-scheduled CLI flag | `[ ]`  |                                                         |
| 384 | Logo click animation (360° spin + sparkle)   | `[ ]`  |                                                         |
| 385 | Engine spinner animation next to START button | `[ ]`  |                                                        |
| 386 | Performance mode — software render enabled   | `[ ]`  |                                                         |
| 387 | Settings gear red dot when update available  | `[ ]`  |                                                         |
| 388 | Unsaved changes guard on navigation away     | `[ ]`  |                                                         |
| 389 | ComboBox scroll-wheel prevention             | `[ ]`  |                                                         |


---

## Phase 2: Profile Management 👤

| #   | Test                                             | Status | Notes                                                                                |
| --- | ------------------------------------------------ | ------ | ------------------------------------------------------------------------------------ |
| 7   | Default profile exists on fresh install          | `[ ]`  | working                                                                              |
| 8   | Create a new profile                             | `[x]`  | ✅ Working                                                                            |
| 9   | Create profile with duplicate name → error shown | `[ ]`  | working                                                                              |
| 10  | Create profile with reserved name → blocked      | `[ ]`  | working                                                                              |
| 11  | Rename a profile                                 | `[ ]`  | working                                                                              |
| 12  | Delete a profile                                 | `[ ]`  | working                                                                              |
| 13  | Switch between profiles — macro list updates     | `[x]`  | ✅ Working                                                                            |
| 14  | Move a macro card from Profile A → Profile B     | `[ ]`  | working                                                                              |
| 15  | Profile with 0 macros shows empty state          | `[ ]`  | working                                                                              |
| 390 | Profile icon picker (emoji grid + custom input)  | `[ ]`  |                                                                                      |
| 391 | Built-in profiles cannot be renamed/deleted       | `[ ]`  |                                                                                      |
| 392 | Profile deletion cascades (hotkeys + macros)     | `[ ]`  |                                                                                      |
| 393 | Profile running indicator (green dot)            | `[ ]`  |                                                                                      |
| 394 | Profile drag-drop target for macros              | `[ ]`  |                                                                                      |
| 395 | Collapsible profiles section in sidebar          | `[ ]`  |                                                                                      |
| 396 | Profile auto-switch on delete                    | `[ ]`  |                                                                                      |
| 397 | Profile icon auto-assignment by name             | `[ ]`  |                                                                                      |
| 398 | Rename reuses ProfileCreationWindow in edit mode | `[ ]`  |                                                                                      |


---

## Phase 3: Macro Library (Dashboard) 📚

| #   | Test                                                     | Status | Notes |
| --- | -------------------------------------------------------- | ------ | ----- |
| 16  | Macro library loads all saved macros                     | `[ ]`  | yes   |
| 17  | Macro cards display name, icon, hotkey                   | `[ ]`  | yes   |
| 18  | Double-click macro card → opens in editor                | `[ ]`  | yes   |
| 19  | Create new macro from dashboard                          | `[ ]`  | yes   |
| 20  | Duplicate a macro                                        | `[ ]`  | yes   |
| 21  | Delete a macro — fully removed from list                 | `[ ]`  | yes   |
| 22  | Rename a macro — updates everywhere                      | `[ ]`  | yes   |
| 23  | Assign emoji icon to a macro                             | `[ ]`  | yes   |
| 24  | Enable/disable macro toggle switch                       | `[ ]`  | yes   |
| 25  | Disabled macro still appears in list but doesn't trigger | `[ ]`  | yes   |
| 26  | Master ON/OFF toggle — disables all macros at once       | `[ ]`  | YES   |
| 27  | Master toggle turns back ON — re-enables active macros   | `[ ]`  | yes   |
| 28  | Hotkey conflict detection — red border shown             | `[ ]`  | yes   |
| 29  | Visual keyboard shows correct bound hotkeys              | `[ ]`  | yes   |
| 30  | Visual keyboard updates when modifier keys held          | `[ ]`  | yes   |
| 399 | Favorites filter toggle                                  | `[ ]`  |       |
| 400 | Favorites toggle on macro card                           | `[ ]`  |       |
| 401 | Tray favorites menu (up to 6 favorites)                 | `[ ]`  |       |
| 402 | Macro execution from tray context menu                   | `[ ]`  |       |
| 403 | Toast notification system (success/warning/error/info)   | `[ ]`  |       |
| 404 | Action categories (File Launchers, App Bound, etc.)     | `[ ]`  |       |
| 405 | Individual action enable/disable toggles                | `[ ]`  |       |
| 406 | Target capture system (file browse, window picker, etc.)| `[ ]`  |       |
| 407 | Hotkey binding with visual "Listening..." state          | `[ ]`  |       |
| 408 | Reset hotkey / Reset path buttons                        | `[ ]`  |       |
| 409 | Engine toggle from library (Start/Stop)                  | `[ ]`  |       |
| 410 | Macro hotkey badges in sidebar                           | `[ ]`  |       |

---

## Phase 4: Custom Actions (Dashboard) ⚡

| #   | Test                                                | Status | Notes |
| --- | --------------------------------------------------- | ------ | ----- |
| 31  | Add a File Launcher custom action                   | `[ ]`  | ok    |
| 32  | Add a Custom Keystroke action                       | `[ ]`  | ok    |
| 33  | Add an App Switcher action                          | `[ ]`  | ok    |
| 34  | File Launcher opens the correct file/app            | `[ ]`  | ok    |
| 35  | Custom Keystroke fires the correct key combo        | `[ ]`  | ok    |
| 36  | App Switcher switches to the correct app            | `[ ]`  | ok    |
| 37  | Double-click custom action card name → inline edit  | `[ ]`  | ok    |
| 38  | Press Enter after editing name → saves              | `[ ]`  | ok    |
| 39  | Press Escape during name edit → cancels             | `[ ]`  | ok    |
| 40  | Enable/disable toggle on custom action card         | `[ ]`  | ok    |
| 41  | Cycle Through Windows setting works on App Switcher | `[ ]`  | ok    |
| 411 | Add Custom Macro action                             | `[ ]`  |       |
| 412 | Remove action card (X button)                       | `[ ]`  |       |
| 413 | Customizable icon/emoji per card                    | `[ ]`  |       |
| 414 | App Switcher: Window Action (Activate vs Close)     | `[ ]`  |       |
| 415 | App Switcher: Minimize if Active                    | `[ ]`  |       |
| 416 | App Switcher: Strict Instance Tracking              | `[ ]`  |       |
| 417 | App Switcher: Restore Window Size                   | `[ ]`  |       |
| 418 | Show macro name on trigger (ShowTriggerFeedback)    | `[ ]`  |       |
| 419 | Move to Profile (inline button)                     | `[ ]`  |       |
| 420 | Hotkey conflict detection (red border + reason)     | `[ ]`  |       |

---

## Phase 5: App Scoping (Targeting) 🎯

| #   | Test                                                      | Status | Notes    |
| --- | --------------------------------------------------------- | ------ | -------- |
| 42  | Open settings gear on a macro card                        | `[ ]`  | ok       |
| 43  | Set scope to "Active Everywhere" — triggers globally      | `[ ]`  | ok       |
| 44  | Set scope to "Include Only" — triggers only in target app | `[ ]`  | ok       |
| 45  | Set scope to "Exclude Target" — blocked in target app     | `[ ]`  | ok       |
| 46  | Clear scoping target — reverts to global                  | `[ ]`  | ok       |
| 47  | Scope tag visible on card (e.g. "Included: notepad.exe")  | `[ ]`  | ok       |
| 421 | Capture App (3-second countdown)                           | `[ ]`  |          |
| 422 | Capture from List (running processes dropdown)             | `[ ]`  |          |
| 423 | Remove individual app pills (X button)                     | `[ ]`  |          |
| 424 | Multiple target apps (comma-separated pills)              | `[ ]`  |          |

---
 
## Phase 6: Trigger Modes 🔑

| #   | Test                                                       | Status | Notes                            |
| --- | ---------------------------------------------------------- | ------ | -------------------------------- |
| 48  | Trigger mode: Press (single tap)                           | `[ ]`  | yes                              |
| 49  | Trigger mode: Double Tap                                   | `[ ]`  | yes                              |
| 50  | Trigger mode: Hold Down                                    | `[ ]`  | yes                              |
| 51  | Trigger mode: Release Up                                   | `[ ]`  | yes                              |
| 52  | Trigger mode: Long Press                                   | `[ ]`  | yes                              |
| 53  | Trigger mode: Toggle (2-slot)                              | `[ ]`  | yes                              |
| 54  | Trigger mode: Toggle with 3+ slots                         | `[ ]`  | yes                              |
| 56  | Trigger mode: Schedule (Interval)                          | `[ ]`  | yes                              |
| 57  | Trigger mode: Mobile Remote (visible on phone)             | `[ ]`  | yes                              |
| 58  | Trigger mode: Auto-Detect (Screen Event)                   | `[ ]`  | YES                              |
| 59  | Hotkey assignment dialog opens                             | `[ ]`  | ok                               |
| 60  | Hotkey persists after app restart                          | `[ ]`  | ok                               |
| 425 | Trigger mode: Press & Release (different macros)           | `[ ]`  |                                  |
| 426 | Toggle with 4-slot (A/B/C/D) configuration                | `[ ]`  |                                  |
| 427 | Toggle with 5-slot (A/B/C/D/E) configuration              | `[ ]`  |                                  |
| 428 | Toggle timeout configuration (Quick/Normal/Relaxed/etc.)   | `[ ]`  |                                  |
| 429 | Toggle show state tooltip                                  | `[ ]`  |                                  |
| 430 | Auto-Repeat: Run on Start                                  | `[ ]`  |                                  |
| 431 | Auto-Detect: On Disappear fire mode                        | `[ ]`  |                                  |
| 432 | Auto-Detect: Fire Once & Stop                              | `[ ]`  |                                  |
| 433 | Auto-Detect: Repeat with Cooldown                          | `[ ]`  |                                  |
| 434 | Auto-Detect: UI Element detect mode                        | `[ ]`  |                                  |
| 435 | Auto-Detect: Pixel Color detect mode                       | `[ ]`  |                                  |
| 436 | Auto-Detect: Search scope options (Smart Area/Full/Window) | `[ ]`  |                                  |
| 437 | Auto-Detect: Notification settings (tooltip/sound)         | `[ ]`  |                                  |
| 438 | Auto-Detect: Fire limit (max triggers)                     | `[ ]`  |                                  |
| 439 | Auto-Detect: Match accuracy / Tolerance presets            | `[ ]`  |                                  |
| 440 | Auto-Detect: Poll interval (500ms/1s/5s/10s/custom)       | `[ ]`  |                                  |

---

## Phase 7: Recording 🔴

| #   | Test                                                                                    | Status | Notes           |
| --- | --------------------------------------------------------------------------------------- | ------ | --------------- |
| 62  | Start recording → recording indicator appears                                           | `[x]`  | yes             |
| 63  | Record mouse left click                                                                 | `[x]`  | yes             |
| 64  | Record mouse right click                                                                | `[x]`  | yes             |
| 65  | Record mouse middle click                                                               | `[x]`  | yes             |
| 66  | Record keyboard typing (normal text)                                                    | `[x]`  | yes             |
| 67  | Record keyboard shortcut (Ctrl+C, Alt+Tab, etc.)                                        | `[x]`  | yes             |
| 68  | Record Shift+typing (HELLO) → Smart Shift Detection                                     | `[ ]`  | hidden/disabled |
| 69  | Record mouse drag & drop                                                                | `[x]`  | yes             |
| 70  | Record scroll wheel up                                                                  | `[x]`  | yes             |
| 71  | Record scroll wheel down                                                                | `[x]`  | yes             |
| 72  | Record very fast typing                                                                 | `[x]`  | yes             |
| 73  | Record very fast clicking                                                               | `[x]`  | yes             |
| 74  | Stop recording → timeline order is correct                                              | `[x]`  | yes             |
| 75  | App minimizes to tray during recording                                                  | `[x]`  | yes             |
| 76  | Recording position safety popup shown (mid-macro insert)                                | `[x]`  | yes             |
| 77  | Resume/insert recording from a selected step                                            | `[x]`  | yes             |
| 78  | Record 500+ steps — no lag                                                              | `[ ]`  |                 |
| 79  | Stop recording via shortcut cleanly terminates without capturing the stop hotkey itself | `[x]`  | yes             |
| 80  | Stop recording via UI button cleanly terminates                                         | `[x]`  | yes             |
| 441 | Smart Auto-Delay (configurable delays after clicks/keys)                                | `[ ]`  |                 |
| 442 | Loop Pattern Detection (suggests Repeat block)                                          | `[ ]`  |                 |
| 443 | Recording View Mode Toggle (No Trace / All Trace)                                       | `[ ]`  |                 |
| 444 | Widget Stop Click Detection (strips trailing movement)                                  | `[ ]`  |                 |
| 445 | Empty Recording Cleanup (removes stray steps)                                           | `[ ]`  |                 |
| 446 | Smart Scroll Bundling (consecutive scrolls merged)                                      | `[ ]`  |                 |
| 447 | Initial CapsLock State Capture                                                          | `[ ]`  |                 |
| 448 | Auto-Stop/Restart Engine during recording                                               | `[ ]`  |                 |

---

## Phase 8: Timeline Editor ✏️

| #   | Test                                                             | Status | Notes                                 |
| --- | ---------------------------------------------------------------- | ------ | ------------------------------------- |
| 81  | Smart View grouping displays correctly                           | `[-]`  | Obsolete (hidden/disabled)            |
| 82  | Switch Smart View ↔ Raw View — data preserved                    | `[-]`  | Obsolete (hidden/disabled)            |
| 83  | Switch to Extreme Mini View (DataGrid)                           | `[-]`  | Obsolete (hidden/disabled)            |
| 84  | Cycle timeline layout (Normal → Compact → List)                  | `[-]`  | Obsolete (hidden/disabled)            |
| 85  | Add Mouse block manually (Ctrl+M)                                | `[x]`  | ✅ Working                             |
| 86  | Add Keyboard block manually (Ctrl+K)                             | `[x]`  | ✅ Working                             |
| 87  | Add Wait/Delay block (Ctrl+W)                                    | `[x]`  | ✅ Working                             |
| 88  | Add Type Text block (Ctrl+T)                                     | `[x]`  | ✅ Working                             |
| 89  | Add If/Else logic block                                          | `[ ]`  |                                       |
| 90  | Add Loop/Repeat block                                            | `[ ]`  |                                       |
| 91  | Add Group block                                                  | `[ ]`  |                                       |
| 92  | Add Image Search block                                           | `[ ]`  |                                       |
| 93  | Add Pixel Search block                                           | `[ ]`  |                                       |
| 94  | Add Mouse Trace block                                            | `[ ]`  |                                       |
| 95  | Add Call Macro (sub-macro) block                                 | `[ ]`  |                                       |
| 96  | Add User Input block                                             | `[ ]`  |                                       |
| 97  | Add System Sound block                                           | `[ ]`  |                                       |
| 98  | Add Notification block                                           | `[ ]`  |                                       |
| 99  | Add Window Action block                                          | `[ ]`  |                                       |
| 100 | Add UI Element block                                             | `[ ]`  |                                       |
| 101 | Add Wait for Key block                                           | `[ ]`  |                                       |
| 102 | Add WaitUntil block (wait for image/pixel)                       | `[ ]`  |                                       |
| 103 | Delete block (Del key)                                           | `[x]`  | ✅ Working                             |
| 104 | Duplicate block (Ctrl+D)                                         | `[x]`  | ✅ Working                             |
| 105 | Undo (Ctrl+Z)                                                    | `[x]`  | ✅ Working                             |
| 106 | Redo (Ctrl+Y)                                                    | `[x]`  | ✅ Working                             |
| 107 | Drag & drop reorder blocks                                       | `[!]`  | working                               |
| 108 | Move block up (Ctrl+↑)                                           | `[x]`  | ✅ Working                             |
| 109 | Move block down (Ctrl+↓)                                         | `[x]`  | ✅ Working                             |
| 110 | Lasso-select multiple blocks at once                             | `[ ]`  | I think I don't have the feature yet. |
| 111 | Multi-select then delete all selected                            | `[ ]`  | ok                                    |
| 112 | Edit mouse coordinates on a step                                 | `[ ]`  |                                       |
| 113 | Edit text value on a Type Text step                              | `[ ]`  |                                       |
| 114 | Step gear ⚙ popup opens on a mouse step                          | `[ ]`  |                                       |
| 115 | Step gear: enable humanization → gear turns red                  | `[ ]`  |                                       |
| 116 | Step gear: coordinate mode options work                          | `[ ]`  |                                       |
| 117 | Press/Hold/Release tri-state button cycles on click              | `[ ]`  |                                       |
| 118 | Compact mode: card collapsed by default — double-click to expand | `[-]`  | Obsolete (hidden/disabled)            |
| 119 | List mode: card collapsed — double-click to expand               | `[-]`  | Obsolete (hidden/disabled)            |
| 120 | Compact/List mode: summary text shown on collapsed card          | `[-]`  | Obsolete (hidden/disabled)            |
| 121 | Drag step into a Group block                                     | `[ ]`  |                                       |
| 122 | Drag step into an If/Else block                                  | `[ ]`  |                                       |
| 123 | Collapse a Group block → block count badge shown                 | `[ ]`  |                                       |
| 124 | Double-click Group name → inline rename                          | `[ ]`  |                                       |
| 125 | Double-click Loop name → inline rename                           | `[ ]`  |                                       |
| 126 | Unsaved changes guard: navigate away → warning shown             | `[ ]`  |                                       |
| 127 | F1 shortcut cheat sheet opens                                    | `[ ]`  |                                       |
| 128 | Add Action menu — collapse/expand categories                     | `[ ]`  |                                       |
| 129 | Add Action menu — cycle Detailed ↔ Compact view                  | `[-]`  | Obsolete (hidden/disabled)            |
| 130 | Aurora pulse animation on new step insertion                     | `[ ]`  |                                       |
| 131 | Accent color bars toggle (Show/Hide)                             | `[ ]`  |                                       |
| 132 | 500+ steps in timeline — still smooth                            | `[ ]`  |                                       |
| 449 | IsNew block state (suppresses warning dot on new blocks)          | `[ ]`  |                                       |
| 450 | Step Name Duplicate Detection                                    | `[ ]`  |                                       |
| 451 | Safety Warning Display (missing target, long delay, high loop)    | `[ ]`  |                                       |
| 452 | Group Color System (full theming)                                | `[ ]`  |                                       |
| 453 | Group Note (text annotations)                                    | `[ ]`  |                                       |
| 454 | Step Depth Tracking (nesting indentation)                        | `[ ]`  |                                       |
| 455 | LogicIf Variable Modes (VariableEquals/NotEquals)                | `[ ]`  |                                       |
| 456 | SetVariable Block                                                | `[ ]`  |                                       |
| 457 | WaitUntil Block (ImageFound/PixelFound/WindowActive/Exists)      | `[ ]`  |                                       |
| 458 | UI Element Path (hierarchy breadcrumb)                           | `[ ]`  |                                       |
| 459 | UI Screenshot Path                                               | `[ ]`  |                                       |
| 460 | UI Find Mode Options (Exact/SameApp/FindLatest/FindFirst)        | `[ ]`  |                                       |
| 461 | UI Control Type (Button/Edit/CheckBox etc.)                      | `[ ]`  |                                       |
| 462 | FindAllMatches Mode + MatchSelectMode (First/Last)               | `[ ]`  |                                       |
| 463 | Search Cascade Toggles (LastPosition/SmartBox/Window/FullScreen) | `[ ]`  |                                       |
| 464 | Browser Tab Switch (BrowserTabSwitchEnabled/BrowserTabName)      | `[ ]`  |                                       |
| 465 | Popup Mode Options (Checkpoint/Auto-Timeout/Tooltip)             | `[ ]`  |                                       |
| 466 | User Input Types (Text/Dropdown/YesNo)                           | `[ ]`  |                                       |
| 467 | Fast Paste Mode (IsFastPasteMode)                                | `[ ]`  |                                       |
| 468 | LogicIf Nested Structure (ParentId/ChildSteps/ChildStepsFalse)   | `[ ]`  |                                       |
| 469 | Recording Start Marker (IsRecordingStart)                        | `[ ]`  |                                       |

---

## Phase 9: Playback ▶️

| #   | Test                                                    | Status | Notes                      |
| --- | ------------------------------------------------------- | ------ | -------------------------- |
| 133 | Play simple mouse click macro                           | `[x]`  | ✅ Working                  |
| 134 | Play keyboard typing macro                              | `[!]`  | yes                        |
| 135 | Play macro with custom delay/wait step                  | `[ ]`  | yes                        |
| 136 | Play macro with If/Else condition                       | `[ ]`  |                            |
| 137 | Play macro with Loop (correct count)                    | `[ ]`  |                            |
| 138 | Play macro with nested Loop inside Group                | `[ ]`  |                            |
| 139 | Play macro with Call Macro (sub-macro)                  | `[ ]`  |                            |
| 140 | Play macro with Image Search step                       | `[ ]`  |                            |
| 141 | Play macro with Pixel Search step                       | `[ ]`  |                            |
| 142 | Stop macro mid-execution — stops cleanly                | `[ ]`  |                            |
| 143 | Master Kill Switch (double Escape) — stops all          | `[ ]`  |                            |
| 144 | Play very long macro (100+ steps)                       | `[ ]`  |                            |
| 145 | Play macro back-to-back rapidly                         | `[ ]`  |                            |
| 146 | Preview speed: Fast Scroll enabled                      | `[-]`  | Obsolete (hidden/disabled) |
| 147 | Preview speed: Fast Click enabled                       | `[-]`  | Obsolete (hidden/disabled) |
| 148 | Preview speed: Fast Typing enabled                      | `[-]`  | Obsolete (hidden/disabled) |
| 149 | Hardware Input Lock — blocks user input during playback | `[ ]`  |                            |
| 470 | Humanization System (Safe/Normal/Risky/Chaos levels)    | `[ ]`  |                            |
| 471 | Mouse Physics Profiles (Smooth/Instant/Original)        | `[ ]`  |                            |
| 472 | Mouse Movement Style (Smart/AlwaysAnimate)              | `[ ]`  |                            |
| 473 | Mouse Return to Origin                                  | `[ ]`  |                            |
| 474 | Dynamic Offset Capture (runtime offset)                 | `[ ]`  |                            |
| 475 | Smart Retry (MaxRetries + RetryDelayMs)                 | `[ ]`  |                            |
| 476 | Debug Highlights (visual overlay for found elements)    | `[ ]`  |                            |
| 477 | Auto-Launch Missing Windows                             | `[ ]`  |                            |
| 478 | Window Memory (CaptureWindowParameters)                 | `[ ]`  |                            |
| 479 | UI Element Background Mode (UIAutomation InvokePattern) | `[ ]`  |                            |
| 480 | UI Element Scroll Into View                             | `[ ]`  |                            |
| 481 | UI Element Fallback to Coordinates                      | `[ ]`  |                            |
| 482 | UI Element Match by Process                             | `[ ]`  |                            |
| 483 | SetVariable Execution (variable storage)                | `[ ]`  |                            |
| 484 | WaitUntil Execution (polling + timeout)                 | `[ ]`  |                            |
| 485 | Progressive Search Cascade                              | `[ ]`  |                            |
| 486 | Named Target Tracking                                   | `[ ]`  |                            |
| 487 | Step Success State Tracking                             | `[ ]`  |                            |
| 488 | FindText Tolerance (foreground/background)              | `[ ]`  |                            |
| 489 | Recursion Depth Limit (CallMacro 10 levels)             | `[ ]`  |                            |
| 490 | Cursor Release Safety (ReleaseAllHeldInputs)            | `[ ]`  |                            |
| 491 | Execution Semaphore (prevents concurrent execution)     | `[ ]`  |                            |

---

## Phase 10: Mouse Trace 🖱️

| #   | Test                                        | Status | Notes |
| --- | ------------------------------------------- | ------ | ----- |
| 150 | Enable "Record Full Mouse Path" in settings | `[ ]`  |       |
| 151 | Record a macro with mouse movement          | `[ ]`  |       |
| 152 | Play trace: Smooth physics profile          | `[ ]`  |       |
| 153 | Play trace: Instant physics profile         | `[ ]`  |       |
| 154 | Play trace: Original physics profile        | `[ ]`  |       |
| 155 | Speed multiplier: play trace at 2x speed    | `[ ]`  |       |
| 156 | Trace file bundled on export                | `[ ]`  |       |
| 157 | Draw Mouse Path Line visually works during trace replay                | `[ ]`  |       |
| 158 | Coordinates update correctly on screen while drawing path              | `[ ]`  |       |
| 159 | No Trace Mode vs Trace Mode recording works as expected                | `[ ]`  |       |
| 492 | Drag complexity analysis (Simple/Complex classification)              | `[ ]`  |       |
| 493 | Window-relative trace coordinates                                     | `[ ]`  |       |
| 494 | Drag trace HoldDelayMs/ReleaseDelayMs                                 | `[ ]`  |       |

---

## Phase 11: Image Recognition 🔍

| # | Test | Status | Notes |
|---|------|--------|-------|
| 160 | Open Capture Overlay from editor | `[ ]` | |
| 161 | Two-stage capture: click anchor, drag region | `[ ]` | |
| 162 | Smart Window Snapping: one-click captures window | `[ ]` | |
| 163 | Image Search block: Gray mode matching | `[ ]` | |
| 164 | Image Search block: Color mode matching | `[ ]` | |
| 165 | Image Search block: GrayDiff (edge detection) mode | `[ ]` | |
| 166 | Pixel Search block: finds correct pixel color | `[ ]` | |
| 167 | Search scope: Full Screen | `[ ]` | |
| 168 | Search scope: Custom Area | `[ ]` | |
| 169 | Search scope: Window Live (dynamic coordinates) | `[ ]` | |
| 495 | FindAllMatches + MatchSelectMode (First/Last)   | `[ ]` | |
| 496 | Last Known Position Cache                       | `[ ]` | |
| 497 | Smart Retry (MaxRetries + RetryDelayMs)         | `[ ]` | |
| 498 | Debug Highlight overlay                         | `[ ]` | |
| 499 | Fast/Legacy engine toggle (UseFastEngine)       | `[ ]` | |
| 500 | FindText tolerance controls (fg/bg)             | `[ ]` | |
| 501 | Smart Search Box Size                           | `[ ]` | |
| 502 | UIElement capture/interaction (all actions)     | `[ ]` | |
| 503 | Window scope types (WIN_LIVE/WIN_REL/WIN_SMART) | `[ ]` | |
| 504 | FailIfMissing                                  | `[ ]` | |

---

## Phase 12: Import & Export 📦

| # | Test | Status | Notes |
|---|------|--------|-------|
| 170 | Export a macro → .pxmacro file created | `[ ]` | |
| 171 | Import a .pxmacro file → macro appears in library | `[ ]` | |
| 172 | Export macro with mouse trace → .dat file bundled | `[ ]` | |
| 173 | Export macro with image search → image bundled | `[ ]` | |
| 174 | Import on different resolution → coordinates scaled | `[ ]` | |
| 175 | Delete macro used as sub-macro → warning shown | `[ ]` | |
| 176 | Rename macro used as sub-macro → reference updates | `[ ]` | |
| 505 | Export/Import Multiple Macros (.pxprofile)          | `[ ]` | |
| 506 | Hotkey binding preserved on export/import           | `[ ]` | |
| 507 | Missing media warning after import                  | `[ ]` | |
| 508 | Export with Audio files                             | `[ ]` | |
| 509 | Export with UI Screenshot                           | `[ ]` | |
| 510 | Atomic export write (temp+rename)                   | `[ ]` | |

---

## Phase 13: Text Snippets ✍️

| # | Test | Status | Notes |
|---|------|--------|-------|
| 177 | Text Snippets page loads | `[ ]` | |
| 178 | Add a new snippet | `[ ]` | |
| 179 | Set snippet trigger (e.g. `.hello`) | `[ ]` | |
| 180 | Set snippet output text | `[ ]` | |
| 181 | Enable snippet toggle | `[ ]` | |
| 182 | Type trigger in any app → snippet expands | `[ ]` | |
| 183 | Trigger with "Instant" mode — no spacebar needed | `[ ]` | |
| 184 | Trigger with "Space" mode — spacebar confirms | `[ ]` | |
| 185 | Disable snippet → trigger no longer expands | `[ ]` | |
| 186 | Delete snippet → removed from list | `[ ]` | |
| 187 | Duplicate snippet via context menu | `[ ]` | |
| 188 | Drag to reorder snippets | `[ ]` | |
| 189 | Search snippets by name or trigger | `[ ]` | |
| 190 | Empty state visual shown when no snippets | `[ ]` | |
| 191 | Smart placeholder: `{cursor}` — cursor positioned inside text | `[ ]` | |
| 192 | Smart placeholder: `{Name}` — popup asks for value | `[ ]` | |
| 193 | Smart placeholder: `{date}` — inserts current date | `[ ]` | |
| 194 | Smart placeholder: `{date:+1d}` — inserts tomorrow's date | `[ ]` | |
| 195 | Smart placeholder: `{time}` — inserts current time | `[ ]` | |
| 196 | Smart placeholder: `{clipboard}` — inserts clipboard text | `[ ]` | |
| 197 | Hamburger menu (☰) opens placeholder submenu | `[ ]` | |
| 198 | Special characters blocked in trigger field | `[ ]` | |
| 199 | Case-insensitive matching (settings toggle) | `[ ]` | |
| 511 | {clipboard:trim} and {clipboard:upper}       | `[ ]` | |
| 512 | {date:dddd} (Day of Week)                   | `[ ]` | |
| 513 | {time:HH:mm} (24-hour time)                 | `[ ]` | |
| 514 | {Choose:Label:Opt1|Opt2} (Multi-Choice)     | `[ ]` | |
| 515 | {YourLabel} (Fill-in Blank custom labels)   | `[ ]` | |
| 516 | App Scope per snippet (Global/Include/Exclude)| `[ ]` | |
| 517 | Per-snippet sound toggle                     | `[ ]` | |
| 518 | Global Sound enabled toggle                  | `[ ]` | |
| 519 | Trigger conflict auto-disable                | `[ ]` | |
| 520 | Onboarding guide (3-step first visit)        | `[ ]` | |
| 521 | Expansion count tracking                     | `[ ]` | |
| 522 | Caret autocomplete popup for { variables     | `[ ]` | |

---

## Phase 14: Settings Dashboard ⚙️

### Mouse & Keyboard
| # | Test | Status | Notes |
|---|------|--------|-------|
| 200 | Record Full Mouse Path toggle — saves | `[ ]` | |
| 201 | Mouse Coordinates — shows real-time cursor position | `[ ]` | |
| 202 | Keyboard Input toggle — enables/disables recording keyboard | `[ ]` | |
| 203 | Extreme Paste Speed — new Type Text blocks default to paste mode | `[ ]` | |
| 204 | Hardware Input Lock — blocks input during playback | `[ ]` | |

### Engine
| # | Test | Status | Notes |
|---|------|--------|-------|
| 205 | Auto-Start Engine — AHK engine starts on app launch | `[ ]` | |
| 206 | Auto-Reload Engine — script recompiles on changes | `[ ]` | |
| 207 | Undo History Depth — change to 25/50/100/500 | `[ ]` | |
| 208 | Click Bundling Preset — changes merge threshold | `[ ]` | |

### Appearance
| # | Test | Status | Notes |
|---|------|--------|-------|
| 209 | App UI Zoom 80% — UI scales smaller | `[ ]` | |
| 210 | App UI Zoom 110% — UI scales larger | `[ ]` | |
| 211 | Performance Mode ON — disables GPU effects | `[ ]` | |
| 212 | Magnifier zoom level setting (1x–30x) | `[ ]` | |
| 213 | Magnifier pre-capture delay setting | `[ ]` | |
| 214 | Magnifier color palette options | `[ ]` | |
| 215 | Magnifier style: Solid vs Dashed | `[ ]` | |
| 216 | Magnifier shape: Square vs Circle | `[ ]` | |

### General
| # | Test | Status | Notes |
|---|------|--------|-------|
| 217 | Minimize to System Tray toggle | `[ ]` | |
| 218 | Humanize Timing toggle — random delay variation added | `[ ]` | |
| 219 | Smart Sensitivity slider | `[ ]` | |
| 220 | Advanced Trigger Combos — enables mouse triggers + keyboard combos (Tab+A, F1+1, ~+B) | `[ ]` | |

### Advanced & Security
| # | Test | Status | Notes |
|---|------|--------|-------|
| 221 | VIP Admin Mode — relaunches with UAC elevation | `[ ]` | |
| 222 | Zero-Latency Override — higher OS priority | `[ ]` | |
| 223 | Master Kill Switch — default Double-Escape stops all | `[ ]` | |
| 224 | Master Kill Switch — custom hotkey works | `[ ]` | |

### Keyboard Manager
| # | Test | Status | Notes |
|---|------|--------|-------|
| 225 | Editor shortcut rebinding — change Undo key | `[ ]` | |
| 226 | Editor shortcut rebinding — conflict detection | `[ ]` | |
| 227 | Show Insert Feedback toggle — neon glow on/off | `[ ]` | |
| 228 | Enable Window Picker Hotkeys toggle | `[ ]` | |
| 229 | Hide to Tray During Recording toggle | `[ ]` | |

### Factory Reset (Danger Zone)
| # | Test | Status | Notes |
|---|------|--------|-------|
| 230 | Delete All Macros — wipes database | `[ ]` | |
| 231 | Clear AI Data — removes stored keys | `[ ]` | |
| 232 | Reset Settings — restores all defaults | `[ ]` | |
| 523 | Safety Confirmations toggle            | `[ ]` | |
| 524 | Snippets Global Sound toggle           | `[ ]` | |
| 525 | Snippets Case Insensitive toggle       | `[ ]` | |
| 526 | Fast Paste Text Mode toggle            | `[ ]` | |
| 527 | Cancel Mode dropdown                   | `[ ]` | |
| 528 | Show Floating Stop Button toggle       | `[ ]` | |
| 529 | Show Favorites in Tray Menu            | `[ ]` | |
| 530 | Auto Create Virtual Desktop            | `[ ]` | |
| 531 | Smart Image Search toggle              | `[ ]` | |
| 532 | Auto Flow Image Capture toggle         | `[ ]` | |
| 533 | Enable Capture Size Lock toggle        | `[ ]` | |
| 534 | Use Last Known Position Cache          | `[ ]` | |
| 535 | Always On Top Border (Enable/Color/Thickness) | `[ ]` | |
| 536 | Use App Name First toggle              | `[ ]` | |
| 537 | Auto Launch Missing Windows            | `[ ]` | |
| 538 | Kill Switch Notification toggle        | `[ ]` | |
| 539 | Drag Grace Zone Distance presets        | `[ ]` | |
| 540 | Capture Mouse Position toggle          | `[ ]` | |
| 541 | Capture Keyboard Input toggle          | `[ ]` | |
| 542 | Capture Window Switches toggle         | `[ ]` | |
| 543 | Capture Window Moves toggle            | `[ ]` | |
| 544 | Auto Capture Window Size               | `[ ]` | |
| 545 | Window Switch Sensitivity              | `[ ]` | |
| 546 | Hold Delay Preset                      | `[ ]` | |
| 547 | Mouse Movement Style (Smart/AlwaysAnimate) | `[ ]` | |
| 548 | Playback Feel Mode (Smooth/Instant/Original) | `[ ]` | |
| 549 | Import/Export Single Macro             | `[ ]` | |
| 550 | Import All / Export All (Backup Profile) | `[ ]` | |
| 551 | Editor Shortcuts Reset                 | `[ ]` | |
| 552 | Promo Code Redemption                  | `[ ]` | |
| 553 | Account Section (Email/Tier/Days/Logout)| `[ ]` | |
| 554 | Theme Selection (Dark/Light)           | `[ ]` | |
| 555 | Update Download Progress bar           | `[ ]` | |
| 556 | Timeline Settings (SmartMode/StepFilter/InstantRecord) | `[ ]` | |
| 557 | Minimize App on Preview                | `[ ]` | |
| 558 | Reset Stop Button Position             | `[ ]` | |

---

## Phase 15: AI Assistant 🤖

| # | Test | Status | Notes |
|---|------|--------|-------|
| 233 | AI panel opens from START button | `[ ]` | |
| 234 | AI panel closes correctly | `[ ]` | |
| 235 | Chat mode: send a message — AI responds | `[ ]` | |
| 236 | Chat mode: retry failed message | `[ ]` | |
| 237 | Chat mode: copy assistant message to clipboard | `[ ]` | |
| 238 | Build mode: describe a macro in English | `[ ]` | |
| 239 | Build mode: AI generates macro JSON | `[ ]` | |
| 240 | Build mode: "Inject Macro" button saves macro to library | `[ ]` | |
| 241 | Daily quota counter shows remaining generations | `[ ]` | |
| 242 | AI works offline fallback (model cycling) | `[ ]` | |
| 243 | AI chat history persists after app restart | `[ ]` | |
| 244 | Model selection in settings changes AI model | `[ ]` | |
| 245 | Easter egg: type "who made this" → achievement popup | `[ ]` | |
| 559 | AI Panel BETA badge                                  | `[ ]` | |
| 560 | Clear Chat History button                           | `[ ]` | |
| 561 | Chat history 24-hour auto-cleanup                   | `[ ]` | |
| 562 | Smart Context Injection (editor context to AI)      | `[ ]` | |
| 563 | Shorthand Syntax Parser (FastSteps format)          | `[ ]` | |
| 564 | Auto-inject in Build Mode                           | `[ ]` | |
| 565 | Append Mode (add to existing macro)                 | `[ ]` | |
| 566 | Live Build Simulation animation                     | `[ ]` | |
| 567 | Auto-Save + Hotkey Bypass                           | `[ ]` | |

---

## Phase 16: Mobile Remote 📱

| #   | Test                                                  | Status | Notes |
| --- | ----------------------------------------------------- | ------ | ----- |
| 246 | Enable remote server in settings                      | `[ ]`  | yes   |
| 247 | QR code or URL shown to connect phone                 | `[ ]`  | yes   |
| 248 | Connect phone via same WiFi                           | `[ ]`  | yes   |
| 249 | PIN auth — enter correct PIN → authenticated          | `[ ]`  | yes   |
| 250 | PIN auth — 3 wrong PINs → lockout (30s)               | `[ ]`  | yes   |
| 251 | Macro buttons visible on phone                        | `[ ]`  | yes   |
| 252 | Trigger macro from phone → runs on PC                 | `[ ]`  | yes   |
| 253 | Stop all macros button on phone                       | `[ ]`  | yes   |
| 254 | Volume control from phone                             | `[ ]`  | yes   |
| 255 | Soundboard: upload a sound from phone                 | `[ ]`  | yes   |
| 256 | Soundboard: play a sound from phone                   | `[ ]`  | yes   |
| 257 | Soundboard: delete a sound from phone                 | `[ ]`  | yes   |
| 258 | Long-press button → Quick Actions Ring appears        | `[ ]`  |       |
| 259 | Quick Actions Ring: Edit → opens bottom sheet         | `[ ]`  |       |
| 260 | Quick Actions Ring: Color → cycles color              | `[ ]`  |       |
| 261 | Quick Actions Ring: Size → cycles 1×1 → 2×1 → 2×2     | `[ ]`  |       |
| 262 | Quick Actions Ring: Move → enters drag reorder mode   | `[ ]`  |       |
| 263 | Quick Actions Ring: Del → removes from grid           | `[ ]`  |       |
| 264 | Button customization: change emoji                    | `[ ]`  |       |
| 265 | Button customization: upload custom image             | `[ ]`  |       |
| 266 | Button customization: change color                    | `[ ]`  |       |
| 267 | Button customization: rename button                   | `[ ]`  |       |
| 268 | Button customization: toggle show/hide name           | `[ ]`  |       |
| 269 | Drag & drop reorder buttons on phone                  | `[ ]`  |       |
| 270 | Add new page — page indicator dot appears             | `[ ]`  |       |
| 271 | Swipe left/right to change pages                      | `[ ]`  |       |
| 272 | Remove a page (not last page)                         | `[ ]`  |       |
| 273 | Add macro to empty grid slot                          | `[ ]`  |       |
| 274 | Remove macro from grid slot                           | `[ ]`  |       |
| 275 | Disconnect phone → reconnect gracefully               | `[ ]`  |       |
| 276 | Change port in settings — server restarts on new port | `[ ]`  |       |
| 277 | Controller mode: screenshot visible on phone          | `[ ]`  |       |
| 278 | Controller mode: tap to click on phone                | `[ ]`  |       |
| 568 | Remote Access (Cloudflare Tunnel)                    | `[ ]`  |       |
| 569 | Remote Access Status Text                            | `[ ]`  |       |
| 570 | Cloudflare Tunnel auto-download                      | `[ ]`  |       |
| 571 | Per-IP brute-force rate limiting                     | `[ ]`  |       |
| 572 | Session token authentication                         | `[ ]`  |       |
| 573 | Controller Page (separate route)                     | `[ ]`  |       |
| 574 | Controller Trackpad (touch mouse movement)           | `[ ]`  |       |
| 575 | Controller Keyboard (full keyboard)                  | `[ ]`  |       |
| 576 | Controller Scroll API                                | `[ ]`  |       |
| 577 | Controller Mouse Move API                            | `[ ]`  |       |
| 578 | Remote Settings Panel on Mobile                      | `[ ]`  |       |
| 579 | PWA Manifest (Add to Home Screen)                    | `[ ]`  |       |
| 580 | Soundboard toggle (double-tap to stop)               | `[ ]`  |       |
| 581 | Page Rename                                          | `[ ]`  |       |
| 582 | Tip Banner ("Add to Home Screen")                    | `[ ]`  |       |
| 583 | Landscape auto-rotation CSS                          | `[ ]`  |       |

---

## Phase 17: Auto-Update 🔄

| # | Test | Status | Notes |
|---|------|--------|-------|
| 279 | Update check runs on startup | `[ ]` | |
| 280 | Update available → notification/prompt shown | `[ ]` | |
| 281 | Update launches PowerX_Updater correctly | `[ ]` | |
| 283 | App correctly compares local version against new remote versioning strategy | `[ ]`  |       |
| 284 | Semantic version formatting displays correctly in What's New           | `[ ]`  |       |
| 585 | Background silent download                              | `[ ]`  |       |
| 586 | Download progress tracking                              | `[ ]`  |       |
| 587 | Update Downloaded event                                 | `[ ]`  |       |
| 588 | Dismissed Update Version tracking                       | `[ ]`  |       |
| 589 | Force Update (hard block screen)                        | `[ ]`  |       |
| 590 | Self-healing updater (restores if deleted)              | `[ ]`  |       |

---

## Phase 18: Persistence & Data 💾

| #   | Test                                        | Status | Notes |
| --- | ------------------------------------------- | ------ | ----- |
| 284 | Save macro → persists after app restart     | `[ ]`  | yes   |
| 285 | Delete macro → fully gone after restart     | `[ ]`  | yes   |
| 286 | All settings save and persist after restart | `[ ]`  | yes   |
| 287 | Hotkey bindings persist after restart       | `[ ]`  | yes   |
| 288 | Profile data persists after restart         | `[ ]`  | yes   |
| 289 | Text snippets persist after restart         | `[ ]`  | yes   |
| 290 | AI chat history persists after restart      | `[ ]`  | yes   |
| 291 | Remote customizations persist after restart | `[ ]`  | yes   |
| 292 | App works fully offline (no internet)       | `[ ]`  | yes   |
| 591 | DB corruption recovery + UI notification    | `[ ]`  |       |
| 592 | Config backup/restore on corruption         | `[ ]`  |       |
| 593 | Safe-write config (atomic write)            | `[ ]`  |       |
| 594 | Profile auto-repair/merge                   | `[ ]`  |       |
| 595 | Starter macros seeding (MacrosSeeded flag)  | `[ ]`  |       |
| 596 | Legacy TraceCaptureMode migration           | `[ ]`  |       |
| 597 | LogicIfExperimental→LogicIf migration       | `[ ]`  |       |
| 598 | Capture library stale entry cleanup         | `[ ]`  |       |
| 599 | Orphaned file cleanup                       | `[ ]`  |       |

---

## Phase 19: Easter Eggs 🥚

| #   | Test                                                               | Status | Notes |
| --- | ------------------------------------------------------------------ | ------ | ----- |
| 293 | Easter egg 1: Click version badge 7 times → "The Architect"        | `[ ]`  | yes   |
| 294 | Easter egg 2: Type Konami Code (↑↑↓↓←→←→BA) → "Old School Gamer"   | `[ ]`  | yes   |
| 295 | Easter egg 3: Type "who made this" in AI chat → "The Curious Mind" | `[ ]`  | yes   |
| 296 | Easter egg 4: Name a macro "Maaz" → "Name Dropper"                 | `[ ]`  | yes   |
| 297 | Easter egg 5: Type "powerx" anywhere → "Whisper In The Dark"       | `[ ]`  | yes   |
| 298 | Easter egg 6: Shift+Click the app logo → "The Shift Clicker"       | `[ ]`  | yes   |
| 299 | Achievement popup: golden card, confetti, sound plays              | `[ ]`  | yes   |
| 300 | Achievement counter increments (X / 6 Secrets Found)               | `[ ]`  | yes   |
| 301 | Already-found egg: popup does NOT show again                       | `[ ]`  | yes   |
| 600 | Easter egg persistence (UnlockedEasterEggs list)                  | `[ ]`  |       |
| 601 | Easter egg progress bar (UnlockedCount / Total)                   | `[ ]`  |       |
| 602 | "powerx" detected globally (any keypress, 3s buffer)             | `[ ]`  |       |

---

## Phase 20: Edge Cases & Stability 🔥

| #   | Test                                                                                                                                                                                | Status | Notes |
| --- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------ | ----- |
| 302 | Empty macro → play (no crash)                                                                                                                                                       | `[ ]`  |       |
| 303 | Macro with only 1 block → play correctly                                                                                                                                            | `[ ]`  |       |
| 304 | Delete all blocks → Undo → all restored                                                                                                                                             | `[ ]`  |       |
| 305 | Close app during recording                                                                                                                                                          | `[ ]`  |       |
| 306 | Close app during playback                                                                                                                                                           | `[ ]`  |       |
| 307 | Same hotkey on two macros → conflict handled                                                                                                                                        | `[ ]`  |       |
| 308 | Very long macro name (50+ chars) → UI handles it                                                                                                                                    | `[ ]`  |       |
| 309 | Rapid create/delete macros — no crash                                                                                                                                               | `[ ]`  |       |
| 310 | App restart after Factory Reset — clean state                                                                                                                                       | `[ ]`  |       |
| 311 | Recording 500+ steps — still smooth after stop                                                                                                                                      | `[ ]`  |       |
| 312 | Navigate to Settings → back to Library — state preserved                                                                                                                            | `[ ]`  |       |
| 313 | Navigate Settings → Editor → Library — no crashes                                                                                                                                   | `[ ]`  |       |
| 314 | Macro with mouse trace on different resolution                                                                                                                                      | `[ ]`  |       |
| 315 | **[Editor]** Drag a parent Group into its own child → blocked (no circular ref)                                                                                                     | `[ ]`  |       |
| 316 | **[Editor]** Undo immediately after recording stops → recording reversed                                                                                                            | `[ ]`  |       |
| 317 | **[Editor]** Navigation guard → press CANCEL → stays in editor, changes kept                                                                                                        | `[ ]`  |       |
| 318 | **[Editor]** Navigation guard → press EXIT → discards changes, navigates away                                                                                                       | `[ ]`  |       |
| 319 | **[Editor]** Delete a Group block that has children → children also removed                                                                                                         | `[ ]`  |       |
| 320 | **[Editor]** Undo depth limit reached (e.g. 25-level) → oldest state dropped gracefully                                                                                             | `[ ]`  |       |
| 321 | **[Editor]** Custom rebound editor shortcut works inside the editor (e.g. rebound Undo key)                                                                                         | `[ ]`  |       |
| 322 | **[Editor]** Insert recording mid-macro (step selected) → safety popup shown first                                                                                                  | `[ ]`  |       |
| 323 | **[Editor]** Undo/Redo after drag-reorder → correct order restored                                                                                                                  | `[ ]`  |       |
| 324 | **[Keyboard Combo]** Advanced toggle OFF → single keys capture instantly (no delay)                                                                                                 | `[ ]`  |       |
| 325 | **[Keyboard Combo]** Advanced toggle ON → hold Tab then press A → captures `Tab & a`                                                                                                | `[ ]`  |       |
| 326 | **[Keyboard Combo]** Advanced toggle ON → hold F1 then press 1 → captures `F1 & 1`                                                                                                  | `[ ]`  |       |
| 327 | **[Keyboard Combo]** Advanced toggle ON → hold `` ` `` then press B → captures `` ` & b``                                                                                           | `[ ]`  |       |
| 328 | **[Keyboard Combo]** Advanced toggle ON → single key press+release → captures single key                                                                                            | `[ ]`  |       |
| 329 | **[Keyboard Combo]** Combo macro compiles to AHK `~Tab & a::` format and runs correctly                                                                                             | `[ ]`  |       |
| 330 | **[Toggle Initialization]** Toggle macro fires perfectly on the very first click, even if placed far down the script list (auto-execute boundary test)                              | `[ ]`  |       |
| 331 | **[Auto-Repeat Scoping]** Auto-Repeat macro strictly respects Application Scope settings (Include/Exclude) and suspends automatically when a disallowed app is focused              | `[ ]`  |       |
| 332 | **[Null Safety]** Triggering timeline commands (Remove/Duplicate/Move/Nest) with an empty or invalid step selection does not crash the app (tests `MacroEditorViewModel.Core.cs`)   | `[ ]`  |       |
| 333 | **[UI Thread Blocking]** Stopping a massive recording (500+ steps) does not freeze the UI while post-processing runs                                                                | `[ ]`  |       |
| 334 | **[Memory Allocation]** `DetectLoopPattern` handles large step counts without excessive memory spiking or frame drops                                                               | `[ ]`  |       |
| 335 | **[Data Loss]** Changing `TriggerMode` via JSON deserialization or bulk updates does not accidentally trigger `File.Delete` on valid user images                                    | `[ ]`  |       |
| 336 | **[JSON Parsing Order]** Saved `TriggerClickCount` and `TriggerDuration` settings are correctly loaded and not overwritten by property setters enforcing default values during load | `[ ]`  |       |
| 337 | **[Thread Affinity]** Loading macros on a background thread does not crash the app with an `InvalidOperationException` due to un-frozen `SolidColorBrush` instances in `MacroStep`  | `[ ]`  |       |
| 338 | **[Startup Crash]** `ActionItem.ScheduleUseTaskScheduler` safely handles null `ConfigManager.Current` states when accessed during early app initialization                          | `[ ]`  |       |
| 339 | **[Data Overwrite]** Custom user text inside Wait blocks is preserved and not overwritten by `AutoUpdateWaitForKeyMessage()` during JSON loading                                    | `[ ]`  |       |
| 603 | **[CallMacro Recursion]** Depth limit (10 levels) blocks circular calls with warning dialog                                                               | `[ ]`  |       |
| 604 | **[CallMacro Rename]** Renaming a macro updates all CallMacro references                                       | `[ ]`  |       |
| 605 | **[LogicIf Source Disabled]** If source is disabled, entire block is skipped                                   | `[ ]`  |       |
| 606 | **[CallMacro Target Missing]** Warning dot when target macro deleted                                           | `[ ]`  |       |
| 607 | **[Database Connection Pooling]** Pooling=True;Cache=Shared in connection string                              | `[ ]`  |       |

---

## Phase 21: If/Else Logic Block 🔀

| #   | Test                                                                                                                                                                                | Status | Notes |
| --- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------ | ----- |
| 340 | **[Put a step in the YES box]** Drag any action (like a mouse click) and drop it into the green YES box. Verify it goes inside.                                                    | `[ ]`  |       |
| 341 | **[Put a step in the NO box]** Drag any action (like a mouse click) and drop it into the red NO box. Verify it goes inside.                                                        | `[ ]`  |       |
| 342 | **[Take a step out of the YES/NO boxes]** Drag an action out of the YES or NO box and drop it back onto the main timeline. Verify it moves out.                                    | `[ ]`  |       |
| 343 | **[Check the box colors (Flat design)]** Check that the YES box has a flat solid green border and the NO box has a flat solid red border (no gradients or shading).                | `[ ]`  |       |
| 344 | **[Test YES branch when Image is found]** Capture an image. Put an action in the YES box. Play the macro when the image is visible on screen. Check if the YES action runs.        | `[ ]`  |       |
| 345 | **[Test NO branch when Image is NOT found]** Run the same macro (from Test 344) but hide the image. Put an action in the NO box. Check if the NO action runs instead.               | `[ ]`  |       |
| 346 | **[Test YES branch when Pixel color matches]** Target a pixel color. Put an action in the YES box. Play when the color is visible. Check if the YES action runs.                    | `[ ]`  |       |
| 347 | **[Test YES branch when a UI button is found]** Target a window button. Put an action in the YES box. Play when the button is visible. Check if the YES action runs.                | `[ ]`  |       |
| 348 | **[Test YES/NO branch using a variable value]** Set a variable to a word. Check if YES runs when the word matches, and NO runs when the word does not match.                       | `[ ]`  |       |
| 349 | **[Put a block inside another block]** Drag a second logic block or a loop block and drop it inside the YES/NO box of the first block. Check that they nest properly.               | `[ ]`  |       |
| 350 | **[Delete the logic block]** Click the main logic block card and press the Delete key on your keyboard. Check that the block and everything inside it are deleted.                 | `[ ]`  |       |
| 608 | **[LogicIf: NamedBlockSuccess]** Condition checks if named step succeeded                                      | `[ ]`  |       |
| 609 | **[LogicIf: NamedBlockFailed]** Condition checks if named step failed                                          | `[ ]`  |       |
| 610 | **[LogicIf: AboveStepFailed]** Condition checks if step above failed                                           | `[ ]`  |       |
| 611 | **[LogicIf: VariableEquals]** Compare variable against expected value                                          | `[ ]`  |       |
| 612 | **[LogicIf: VariableNotEquals]** Inverse of VariableEquals                                                     | `[ ]`  |       |
| 613 | **[LogicIf: LogicSource field]** ComboBox to select named step target                                          | `[ ]`  |       |
| 614 | **[LogicIf: IsSourceDisabled toggle]** Disable source reference without deleting                               | `[ ]`  |       |
| 615 | **[LogicIf: Auto-increment StepName]** "If Block N" naming                                                     | `[ ]`  |       |
| 616 | **[LogicIf: ImageTolerance on condition]** Per-block tolerance setting                                         | `[ ]`  |       |

---

## Phase 22: Error Handling & AHK Leak Prevention 🛡️

> **Purpose:** Ensure native AHK error popups NEVER leak to the user. All errors must be caught and shown as PowerX tooltips or logged silently.

| # | Test | Status | Notes |
|---|------|--------|-------|
| 351 | **[FileLauncher]** Try opening a deleted/missing file → "Could not open file" tooltip, no AHK error popup | `[ ]` | |
| 352 | **[MouseClick]** Click on a destroyed/minimized window → "Click failed" tooltip, no AHK error popup | `[ ]` | |
| 353 | **[SendKeys]** Send invalid key name (e.g., `{InvalidKey}`) → "Send failed" tooltip, no AHK error popup | `[ ]` | |
| 354 | **[Scheduled Task]** Remove the task's image file mid-run → error logged, task fails gracefully, no AHK error popup | `[ ]` | |
| 355 | **[ScreenEvent Watcher]** Remove the watched image mid-run → error logged, watcher fails gracefully, no AHK error popup | `[ ]` | |
| 356 | **[SetVariable]** Evaluate a bad/malformed expression → error logged, step fails gracefully, no AHK error popup | `[ ]` | |
| 357 | **[CallMacro]** Call a missing or corrupt macro file → "Macro not found" tooltip, no AHK error popup | `[ ]` | |
| 358 | **[WaitForKey]** Block keyboard access → error logged, step fails gracefully, no AHK error popup | `[ ]` | |
| 359 | **[Notification]** Trigger notification on restricted system → error logged, step fails gracefully, no AHK error popup | `[ ]` | |
| 360 | **[DynamicOffsetCapture]** Trigger offset capture with error → error logged, step fails gracefully, no AHK error popup | `[ ]` | |
| 361 | **[Snippets Script]** Trigger hotstring with bad input → no AHK error popup | `[ ]` | |
| 362 | **[Sandbox Script]** Run with invalid Send key → no AHK error popup | `[ ]` | |
| 363 | **[Preview Mode — ImageSearch]** Trigger ImageSearch with invalid image in single-step → "Error: [message]" tooltip | `[ ]` | |
| 364 | **[Preview Mode — PixelSearch]** Trigger PixelSearch with invalid pixel in single-step → "Error: [message]" tooltip | `[ ]` | |
| 365 | **[Preview Mode — MouseClick]** Trigger MouseClick with bad coordinates in single-step → "Error: [message]" tooltip | `[ ]` | |
| 366 | **[Preview Mode — SendKeys]** Trigger SendKeys with invalid key in single-step → "Error: [message]" tooltip | `[ ]` | |
| 367 | **[Preview Mode — WindowAction]** Trigger WindowAction on missing window in single-step → "Error: [message]" tooltip | `[ ]` | |
| 368 | **[Preview Mode — FileLauncher]** Trigger FileLauncher with deleted file in single-step → "Error: [message]" tooltip | `[ ]` | |
| 369 | **[Build Compilation]** Generate script with syntax error → PowerX error message, not raw AHK output | `[ ]` | |
| 370 | **[Executor Crash]** Force crash mid-execution → stderr logged before respawn | `[ ]` | |
| 371 | **[Error Handler]** Verify OnError handler suppresses default AHK MessageBox in Executor | `[ ]` | |
| 372 | **[Error Handler]** Verify OnError handler suppresses default AHK MessageBox in Listener | `[ ]` | |
| 373 | **[Error Handler]** Verify OnError handler suppresses default AHK MessageBox in Snippets script | `[ ]` | |
| 374 | **[Error Handler]** Verify OnError handler suppresses default AHK MessageBox in Sandbox script | `[ ]` | |

---

## Phase 23: Windows, Dialogs & Utilities 🪟

| # | Test | Status | Notes |
|---|------|--------|-------|
| 617 | ImageStudioWindow — open captured image for editing | `[ ]` | |
| 618 | ImageStudioWindow — adjust mode (Gray/Color/GrayDiff) | `[ ]` | |
| 619 | ImageStudioWindow — preview FindText code | `[ ]` | |
| 620 | ImageStudioWindow — copy code to clipboard | `[ ]` | |
| 621 | ImagePreviewWindow — image preview popup during capture | `[ ]` | |
| 622 | KeyCaptureWindow — standalone hotkey capture dialog | `[ ]` | |
| 623 | InputPromptWindow — user text input during macro execution | `[ ]` | |
| 624 | DropdownPromptWindow — dropdown selection prompt | `[ ]` | |
| 625 | DarkMessageBoxWindow — custom dark-themed confirmations | `[ ]` | |
| 626 | NamingConflictDialog — duplicate name on import | `[ ]` | |
| 627 | CoordinatePickerWindow — crosshair coordinate picker | `[ ]` | |
| 628 | CoordinateEditDialog — manual coordinate edit | `[ ]` | |
| 629 | OffsetCaptureWindow — dynamic offset capture overlay | `[ ]` | |
| 630 | HardwareLockWarningDialog — hardware input lock warning | `[ ]` | |
| 631 | ExportMacroPickerDialog — multi-macro export picker | `[ ]` | |
| 632 | CaptureLibraryWindow — browse saved captures | `[ ]` | |
| 633 | CaptureLibraryWindow — favorite/unfavorite capture | `[ ]` | |
| 634 | CaptureLibraryWindow — delete capture | `[ ]` | |
| 635 | CaptureLibraryWindow — use capture in macro | `[ ]` | |
| 636 | SkeletonOverlay — loading shimmer animation | `[ ]` | |
| 637 | TelemetryService — crash report sending | `[ ]` | |
| 638 | TelemetryService — launch ping | `[ ]` | |
| 639 | NetworkTimeService — network time sync | `[ ]` | |
| 640 | AhkErrorSuppressor — AHK error popup suppression lifecycle | `[ ]` | |
| 641 | HotkeyCaptureHook — low-level keyboard hook | `[ ]` | |
| 642 | Script Library search/filter by name | `[ ]` | |
| 643 | Macro search/filter in dashboard library | `[ ]` | |
| 644 | AlwaysOnTop overlay visual (colored border) | `[ ]` | |
| 645 | Window Picker for scoping (crosshair dialog flow) | `[ ]` | |
| 646 | Promo Code Redemption UI dialog flow | `[ ]` | |
| 647 | Update Download Progress bar visual | `[ ]` | |

---

## Bugs Found 🐛

> List all bugs here with details. Reference the test # for context.

| Bug # | Test # | Description | Severity | Fixed? |
|-------|--------|-------------|----------|--------|
| 1 | #74 | Deleting a combo block (Ctrl+C) splits into smaller blocks (Ctrl Hold + Wait + Ctrl Release) instead of removing entirely | Medium | `[x]` |
| 2 | #77–78 | Scroll preview plays "straight" — needs raw vs smart preview distinction | Low (future) | `[ ]` |
| 3 | #110 | Drag & drop has ~500ms delay on release before showing changes | Low (future) | `[ ]` |
| 4 | #128 | Keyboard typing playback sometimes has delays | Low | `[ ]` |

---

## Testing Progress 📊

| Phase | Total | Passed | Bugs | Skipped |
|-------|-------|--------|------|---------|
| 1. App Startup | 21 | 6 | 0 | 0 |
| 2. Profile Management | 18 | 2 | 0 | 0 |
| 3. Macro Library | 28 | 0 | 0 | 0 |
| 4. Custom Actions | 21 | 0 | 0 | 0 |
| 5. App Scoping | 10 | 0 | 0 | 0 |
| 6. Trigger Modes | 30 | 0 | 0 | 0 |
| 7. Recording | 27 | 9 | 0 | 0 |
| 8. Timeline Editor | 74 | 10 | 1 | 8 |
| 9. Playback | 40 | 1 | 1 | 3 |
| 10. Mouse Trace | 13 | 0 | 0 | 0 |
| 11. Image Recognition | 20 | 0 | 0 | 0 |
| 12. Import | 13 | 0 | 0 | 0 |
| 13. Text Snippets | 35 | 0 | 0 | 0 |
| 14. Settings Dashboard | 71 | 0 | 0 | 0 |
| 15. AI Assistant | 23 | 0 | 0 | 0 |
| 16. Mobile Remote | 50 | 0 | 0 | 0 |
| 17. Auto-Update | 11 | 0 | 0 | 0 |
| 18. Persistence | 18 | 0 | 0 | 0 |
| 19. Easter Eggs | 12 | 0 | 0 | 0 |
| 20. Edge Cases | 43 | 0 | 0 | 0 |
| 21. If/Else Logic Block | 20 | 0 | 0 | 0 |
| 22. Error Handling & AHK Leaks | 24 | 0 | 0 | 0 |
| 23. Windows, Dialogs & Utilities | 31 | 0 | 0 | 0 |
| **Total** | **652** | **28** | **2** | **11** |

---

## Related Pages

- [[planned-features]]
- [[known-issues]]
- [[current-version]]
- [[spec-always-on-top]]