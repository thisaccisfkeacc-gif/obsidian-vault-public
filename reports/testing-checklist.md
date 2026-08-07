---
tags: [status, testing, qa, checklist]
date: 2026-06-10
status: in-progress
---
****

> **Instructions:** Test each item and update status:
> - `[ ]` = Not tested yet
> - `[x]` = ✅ Working correctly
> - `[!]` = 🐛 Bug found (add details in Notes)
> - `[-]` = ⏭️ Skipped / Not applicable

## Phase 9: Playback ▶️

| #   | Test                                                    | Status | Notes                                                                                                                                                   |
| --- | ------------------------------------------------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 139 | Play macro with Call Macro (sub-macro)                  | `[ ]`  | c                                                                                                                                                       |
| 144 | Play very long macro (100+ steps)                       | `[ ]`  |                                                                                                                                                         |
| 145 | Play macro back-to-back rapidly                         | `[ ]`  | Ask AI about this: can I play macro back to back and it will play rapidly without hanging, freezing, or anything?                                       |
| 149 | Hardware Input Lock — blocks user input during playback | `[ ]`  | Not working currently it has some bugs.                                                                                                                 |
| 472 | Mouse Movement Style (Smart/AlwaysAnimate)              | `[ ]`  | Not checked                                                                                                                                             |
| 474 | Dynamic Offset Capture (runtime offset)                 | `[ ]`  | needs improvement because it only works when we preview it. We want something that when the checkbox is ticked, it will immediately ask for the offset. |
| 475 | Smart Retry (MaxRetries + RetryDelayMs)                 | `[ ]`  |                                                                                                                                                         |
| 476 | Debug Highlights (visual overlay for found elements)    | `[ ]`  |                                                                                                                                                         |
| 477 | Auto-Launch Missing Windows                             | `[ ]`  |                                                                                                                                                         |
| 478 | Window Memory (CaptureWindowParameters)                 | `[ ]`  |                                                                                                                                                         |
| 479 | UI Element Background Mode (UIAutomation InvokePattern) | `[ ]`  |                                                                                                                                                         |
| 480 | UI Element Scroll Into View                             | `[ ]`  |                                                                                                                                                         |
| 481 | UI Element Fallback to Coordinates                      | `[ ]`  |                                                                                                                                                         |
| 482 | UI Element Match by Process                             | `[ ]`  |                                                                                                                                                         |
| 483 | SetVariable Execution (variable storage)                | `[ ]`  |                                                                                                                                                         |
| 484 | WaitUntil Execution (polling + timeout)                 | `[ ]`  |                                                                                                                                                         |
| 485 | Progressive Search Cascade                              | `[ ]`  |                                                                                                                                                         |
| 486 | Named Target Tracking                                   | `[ ]`  |                                                                                                                                                         |
| 487 | Step Success State Tracking                             | `[ ]`  |                                                                                                                                                         |
| 488 | FindText Tolerance (foreground/background)              | `[ ]`  |                                                                                                                                                         |
| 489 | Recursion Depth Limit (CallMacro 10 levels)             | `[x]`  | ✅ Verified: macroCallStack ≥ 10 blocks (MacroExecutionService.cs:833) |
| 490 | Cursor Release Safety (ReleaseAllHeldInputs)            | `[x]`  | ✅ Verified: called in finally (MacroExecutionService.cs:684) |
| 491 | Execution Semaphore (prevents concurrent execution)     | `[x]`  | ✅ Verified: SemaphoreSlim(1,1) + WaitAsync(0) + Release (MacroExecutionService.cs:242,577,695) |

---

## Phase 10: Mouse Trace 🖱️

| #   | Test                                                      | Status | Notes |
| --- | --------------------------------------------------------- | ------ | ----- |
| 152 | Play trace: Smooth physics profile                        | `[ ]`  |       |
| 153 | Play trace: Instant physics profile                       | `[ ]`  |       |
| 154 | Play trace: Original physics profile                      | `[ ]`  |       |
| 155 | Speed multiplier: play trace at 2x speed                  | `[ ]`  |       |
| 156 | Trace file bundled on export                              | `[ ]`  |       |
| 157 | Draw Mouse Path Line visually works during trace replay   | `[ ]`  |       |
| 158 | Coordinates update correctly on screen while drawing path | `[ ]`  |       |
| 159 | No Trace Mode vs Trace Mode recording works as expected   | `[ ]`  |       |
| 492 | Drag complexity analysis (Simple/Complex classification)  | `[ ]`  |       |
| 493 | Window-relative trace coordinates                         | `[ ]`  |       |
| 494 | Drag trace HoldDelayMs/ReleaseDelayMs                     | `[ ]`  |       |

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

| #   | Test                                                          | Status | Notes                                                                                                                                                                                                                |
| --- | ------------------------------------------------------------- | ------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 178 | Add a new snippet                                             | `[ ]`  | By default it is adding as off. We have to make it on.                                                                                                                                                               |
| 179 | Set snippet trigger (e.g. `.hello`)                           | `[ ]`  | Need a little bit of improvement on the feedback when we highlight/focus the text box of it.                                                                                                                         |
| 180 | Set snippet output text                                       | `[ ]`  | Crashed                                                                                                                                                                                                              |
| 190 | Empty state visual shown when no snippets                     | `[ ]`  | Needs explanation on it                                                                                                                                                                                              |
| 191 | Smart placeholder: `{cursor}` — cursor positioned inside text | `[ ]`  | Not checked                                                                                                                                                                                                          |
| 192 | Smart placeholder: `{Name}` — popup asks for value            | `[ ]`  | Doesn't properly works and has an ugly UI when it asks for the name in window                                                                                                                                        |
| 194 | Smart placeholder: `{date:+1d}` — inserts tomorrow's date     | `[ ]`  | Not checked                                                                                                                                                                                                          |
| 195 | Smart placeholder: `{time}` — inserts current time            | `[ ]`  | Works but it has 24 hr time, like 3:00PM it says 15:00, and I just got to know that I can use both the 12-hour and 24-hour ones and by default the now time is set to 24 so we need to change it to the 12-hour one. |
| 196 | Smart placeholder: `{clipboard}` — inserts clipboard text     | `[ ]`  | Works but seems like it is kind of slow. When used with space mode not sure about the instant mode if it happens there same or not.                                                                                  |
| 197 | Hamburger menu (☰) opens placeholder submenu                  | `[ ]`  | Need fixing because it dynamically shifts its width, but verify if it's true or not.                                                                                                                                 |
| 198 | Special characters blocked in trigger field                   | `[ ]`  | What does it mean?                                                                                                                                                                                                   |
| 199 | Case-insensitive matching (settings toggle)                   | `[ ]`  | I think this setting needs to be removed.                                                                                                                                                                            |
| 511 | {clipboard:trim} and {clipboard:upper}                        | `[ ]`  | explain                                                                                                                                                                                                              |
| 512 | {date:dddd} (Day of Week)                                     | `[ ]`  | explain                                                                                                                                                                                                              |
| 513 | {time:HH:mm} (24-hour time)                                   | `[ ]`  | explain                                                                                                                                                                                                              |
| 514 | {Choose:Label:Opt1                                            |        | explain                                                                                                                                                                                                              |
| 515 | {YourLabel} (Fill-in Blank custom labels)                     | `[ ]`  | explain                                                                                                                                                                                                              |
| 522 | Caret autocomplete popup for { variables                      | `[ ]`  | Needs explanation on it                                                                                                                                                                                              |

---

## Phase 14: Settings Dashboard ⚙️

### Mouse & Keyboard
| #   | Test                                                             | Status | Notes |
| --- | ---------------------------------------------------------------- | ------ | ----- |
| 200 | Record Full Mouse Path toggle — saves                            | `[ ]`  |       |
| 201 | Mouse Coordinates — shows real-time cursor position              | `[ ]`  |       |
| 202 | Keyboard Input toggle — enables/disables recording keyboard      | `[ ]`  |       |
| 203 | Extreme Paste Speed — new Type Text blocks default to paste mode | `[ ]`  |       |
| 204 | Hardware Input Lock — blocks input during playback               | `[ ]`  |       |

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
| 222 | Turbo Engine Mode — priority boosts while macro runs, relaxes ~3s after last step | `[ ]` | |
| 222b | Turbo Engine Mode — OFF (default) leaves engine at Normal priority | `[ ]` | |
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
| 246 | Enable remote server in settings                      | `[x]`  | ✅ Verified: RemoteServerEnabled + service wiring (SettingsDashboardViewModel.cs:903) |
| 247 | QR code or URL shown to connect phone                 | `[x]`  | ✅ Verified: RemoteServerQrCode + ActiveUrl (SettingsDashboardViewModel.cs:1037) |
| 248 | Connect phone via same WiFi                           | `[x]`  | ✅ Verified: HttpListener LAN + tunnel (RemoteServerService.cs) |
| 249 | PIN auth — enter correct PIN → authenticated          | `[x]`  | ✅ Verified: IsAuthorized PIN check (RemoteServerService.cs:596) |
| 250 | PIN auth — 3 wrong PINs → lockout (30s)               | `[x]`  | ✅ Verified: 3 attempts → 30s lockout (RemoteServerService.cs:642-653) |
| 251 | Macro buttons visible on phone                        | `[x]`  | ✅ Verified: GET /api/macros (RemoteServerService.cs:707) |
| 252 | Trigger macro from phone → runs on PC                 | `[x]`  | ✅ Verified: OnActionTriggered → ExecuteMacroAsync (RemoteServerService.cs) |
| 253 | Stop all macros button on phone                       | `[x]`  | ✅ Verified: POST /api/stop (RemoteServerService.cs:1197) |
| 254 | Volume control from phone                             | `[x]`  | ✅ Verified: /api/volume (RemoteServerService.cs:1244) |
| 255 | Soundboard: upload a sound from phone                 | `[x]`  | ✅ Verified: POST /api/soundboard/upload (RemoteServerService.cs:1378) |
| 256 | Soundboard: play a sound from phone                   | `[x]`  | ✅ Verified: POST /api/soundboard/play/ (RemoteServerService.cs:1536) |
| 257 | Soundboard: delete a sound from phone                 | `[x]`  | ✅ Verified: DELETE /api/soundboard/{id} (RemoteServerService.cs:1643) |
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

| #   | Test                                       | Status | Notes             |
| --- | ------------------------------------------ | ------ | ----------------- |
| 290 | AI chat history persists after restart     | `[ ]`  | debatable         |
| 591 | DB corruption recovery + UI notification   | `[ ]`  | Explanation of it |
| 592 | Config backup/restore on corruption        | `[ ]`  | Explanation of it |
| 593 | Safe-write config (atomic write)           | `[ ]`  | Explanation of it |
| 594 | Profile auto-repair/merge                  | `[ ]`  | Explanation of it |
| 595 | Starter macros seeding (MacrosSeeded flag) | `[ ]`  | Explanation of it |
| 596 | Legacy TraceCaptureMode migration          | `[ ]`  | Explanation of it |
| 597 | LogicIfExperimental→LogicIf migration      | `[ ]`  | Explanation of it |
| 598 | Capture library stale entry cleanup        | `[ ]`  | Explanation of it |
| 599 | Orphaned file cleanup                      | `[ ]`  | Explanation of it |
|     |                                            |        |                   |

---

## Phase 19: Easter Eggs 🥚

| #   | Test                                                               | Status | Notes                     |
| --- | ------------------------------------------------------------------ | ------ | ------------------------- |
| 295 | Easter egg 3: Type "who made this" in AI chat → "The Curious Mind" | `[x]`  | ✅ Verified: creator-verb detection → CuriousMind (AIAssistantViewModel.cs:756-758) |
| 296 | Easter egg 4: Name a macro "Maaz" → "Name Dropper"                 | `[ ]`  | I think it has some issue |




---

## Phase 20: Edge Cases & Stability 🔥

| #   | Test                                                                                                                                                                                | Status | Notes |
| --- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------ | ----- |
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
| 317 | **[Editor]** Navigation guard → press CANCEL → stays in editor, changes kept                                                                                                        | `[ s]` |       |
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
| 603 | **[CallMacro Recursion]** Depth limit (10 levels) blocks circular calls with warning dialog                                                                                         | `[x]`  | ✅ Verified (MacroExecutionService.cs:833 + compiler ToolTip path) |
| 604 | **[CallMacro Rename]** Renaming a macro updates all CallMacro references                                                                                                            | `[ ]`  |       |
| 605 | **[LogicIf Source Disabled]** If source is disabled, entire block is skipped                                                                                                        | `[ ]`  |       |
| 606 | **[CallMacro Target Missing]** Warning dot when target macro deleted                                                                                                                | `[ ]`  |       |
| 607 | **[Database Connection Pooling]** Pooling=True;Cache=Shared in connection string                                                                                                    | `[x]`  | ✅ Verified: MacroDatabase.cs:17 |

---

## Phase 21: If/Else Logic Block 🔀

| #   | Test                                                                                                                                                               | Status | Notes |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------ | ----- |


---

## Phase 22: Error Handling & AHK Leak Prevention 🛡️

> **Purpose:** Ensure native AHK error popups NEVER leak to the user. All errors must be caught and shown as PowerX tooltips or logged silently.

| #   | Test                                                                                                                    | Status | Notes |
| --- | ----------------------------------------------------------------------------------------------------------------------- | ------ | ----- |
| 353 | **[SendKeys]** Send invalid key name (e.g., `{InvalidKey}`) → "Send failed" tooltip, no AHK error popup                 | `[ ]`  |       |
| 354 | **[Scheduled Task]** Remove the task's image file mid-run → error logged, task fails gracefully, no AHK error popup     | `[ ]`  |       |
| 355 | **[ScreenEvent Watcher]** Remove the watched image mid-run → error logged, watcher fails gracefully, no AHK error popup | `[ ]`  |       |
| 356 | **[SetVariable]** Evaluate a bad/malformed expression → error logged, step fails gracefully, no AHK error popup         | `[ ]`  |       |
| 357 | **[CallMacro]** Call a missing or corrupt macro file → "Macro not found" tooltip, no AHK error popup                    | `[ ]`  |       |
| 358 | **[WaitForKey]** Block keyboard access → error logged, step fails gracefully, no AHK error popup                        | `[ ]`  |       |
| 359 | **[Notification]** Trigger notification on restricted system → error logged, step fails gracefully, no AHK error popup  | `[ ]`  |       |
| 360 | **[DynamicOffsetCapture]** Trigger offset capture with error → error logged, step fails gracefully, no AHK error popup  | `[ ]`  |       |
| 361 | **[Snippets Script]** Trigger hotstring with bad input → no AHK error popup                                             | `[ ]`  |       |
| 362 | **[Sandbox Script]** Run with invalid Send key → no AHK error popup                                                     | `[ ]`  |       |
| 363 | **[Preview Mode — ImageSearch]** Trigger ImageSearch with invalid image in single-step → "Error: [message]" tooltip     | `[ ]`  |       |
| 364 | **[Preview Mode — PixelSearch]** Trigger PixelSearch with invalid pixel in single-step → "Error: [message]" tooltip     | `[ ]`  |       |
| 365 | **[Preview Mode — MouseClick]** Trigger MouseClick with bad coordinates in single-step → "Error: [message]" tooltip     | `[ ]`  |       |
| 366 | **[Preview Mode — SendKeys]** Trigger SendKeys with invalid key in single-step → "Error: [message]" tooltip             | `[ ]`  |       |
| 367 | **[Preview Mode — WindowAction]** Trigger WindowAction on missing window in single-step → "Error: [message]" tooltip    | `[ ]`  |       |
| 368 | **[Preview Mode — FileLauncher]** Trigger FileLauncher with deleted file in single-step → "Error: [message]" tooltip    | `[ ]`  |       |
| 369 | **[Build Compilation]** Generate script with syntax error → PowerX error message, not raw AHK output                    | `[ ]`  |       |
| 370 | **[Executor Crash]** Force crash mid-execution → stderr logged before respawn                                           | `[ ]`  |       |
| 371 | **[Error Handler]** Verify OnError handler suppresses default AHK MessageBox in Executor                                | `[ ]`  |       |
| 372 | **[Error Handler]** Verify OnError handler suppresses default AHK MessageBox in Listener                                | `[ ]`  |       |
| 373 | **[Error Handler]** Verify OnError handler suppresses default AHK MessageBox in Snippets script                         | `[ ]`  |       |
| 374 | **[Error Handler]** Verify OnError handler suppresses default AHK MessageBox in Sandbox script                          | `[ ]`  |       |

---

## Phase 23: Windows, Dialogs & Utilities 🪟

| #   | Test                                                  | Status | Notes                                                                                                                           |
| --- | ----------------------------------------------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------- |
| 618 | ImageStudioWindow — adjust mode (Gray/Color/GrayDiff) | `[ ]`  | i think i can make them more smarter or remove them cuz nobody gonna use it, i need a proper discussion on it with an AI agent. |




---

## Bugs Found 🐛

> List all bugs here with details. Reference the test # for context.

| Bug # | Test # | Description                                                                                                               | Severity     | Fixed? |
| ----- | ------ | ------------------------------------------------------------------------------------------------------------------------- | ------------ | ------ |
| 1     | #74    | Deleting a combo block (Ctrl+C) splits into smaller blocks (Ctrl Hold + Wait + Ctrl Release) instead of removing entirely | Medium       | `[x]`  |
| 2     | #77–78 | Scroll preview plays "straight" — needs raw vs smart preview distinction                                                  | Low (future) | `[ ]`  |
|       |        |                                                                                                                           |              |        |


---

## Testing Progress 📊

| Phase                            | Total   | Passed | Bugs  | Skipped |
| -------------------------------- | ------- | ------ | ----- | ------- |
| 1. App Startup                   | 21      | 6      | 0     | 0       |
| 2. Profile Management            | 18      | 2      | 0     | 0       |
| 3. Macro Library                 | 28      | 0      | 0     | 0       |
| 4. Custom Actions                | 21      | 0      | 0     | 0       |
| 5. App Scoping                   | 10      | 0      | 0     | 0       |
| 6. Trigger Modes                 | 30      | 0      | 0     | 0       |
| 7. Recording                     | 27      | 9      | 0     | 0       |
| 8. Timeline Editor               | 74      | 10     | 1     | 8       |
| 9. Playback                      | 40      | 4      | 1     | 3       |
| 10. Mouse Trace                  | 13      | 0      | 0     | 0       |
| 11. Image Recognition            | 20      | 0      | 0     | 0       |
| 12. Import                       | 13      | 0      | 0     | 0       |
| 13. Text Snippets                | 35      | 0      | 0     | 0       |
| 14. Settings Dashboard           | 71      | 0      | 0     | 0       |
| 15. AI Assistant                 | 23      | 0      | 0     | 0       |
| 16. Mobile Remote                | 50      | 12     | 0     | 0       |
| 17. Auto-Update                  | 11      | 0      | 0     | 0       |
| 18. Persistence                  | 18      | 0      | 0     | 0       |
| 19. Easter Eggs                  | 12      | 1      | 0     | 0       |
| 20. Edge Cases                   | 43      | 2      | 0     | 0       |
| 21. If/Else Logic Block          | 20      | 0      | 0     | 0       |
| 22. Error Handling & AHK Leaks   | 24      | 0      | 0     | 0       |
| 23. Windows, Dialogs & Utilities | 31      | 0      | 0     | 0       |
| **Total**                        | **652** | **46** | **2** | **11**  |

---

## Related Pages

- [[planned-features]]
- [[known-issues]]
- [[current-version]]
- [[spec-always-on-top]]