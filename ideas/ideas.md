# 💡 Ideas & Task Backlog

> Master list. Completed items go to `completed_ideas.md`. Archived items go to `Archive/`.

---

## 🎯 Section 4: Deep Audit Scanning Strategy & Future Roadmap


---

## 🛠️ Section 3: Compiler, Logic & Engine Task Checklist

- [ ] **Taskbar Position App Switcher:** Switch to open apps by their left-to-right position on the Windows taskbar. Skip pinned apps that aren't running. *(Plan created: [[taskbar-window-switcher-plan]])*
- [ ] **Floating Account & Stats Menu:** Merge Account section and Performance Stats into a single floating overlay with trial info and logout.
- [ ] **Future Subscription Status States:** Add `grace_period`, `suspended`, `canceled` states to `public.subscription_status` Postgres enum.
- [ ] **OAuth Redirect Success Page UI:** Redesign the local HTTP listener's success page (`/token`, `/auth/callback`) to be branded and beautiful.
- [ ] **Window Block Preview — Auto-Restore & Highlight:** When previewing a Window block for a minimized window, auto-restore it, sonar-pulse it, and show a confirmation tooltip. Add a global settings toggle.
- [ ] **Disable Preview Button for Passive-Only Macros:** Grey out the Preview toolbar button when the macro only has standalone search blocks with no follow-up actions.
- [ ] **Image-to-Image Drag and Drop:** Drag from a named Image 1 target to a named Image 2 target. Requires new `DragEndTarget` model property.

---

## 🚀 Section 5: Pre-Launch Deployment Checklist

- [ ] **Dodo Payments Production Switch:** Change Dodo Payments from test mode (`https://test.dodopayments.com`) to live mode (`https://live.dodopayments.com`) in `api/checkout.js` and set live Dodo Product IDs in Vercel environment variables before final launch.

---

## 🛠️ Section 3: Compiler, Logic & Engine Task Checklist

- [ ] **Logic Block "Handled" Visual Indicator:** When a Logic Block is connected to a Search/Window/UIElement step, show a small green dot or chain icon on the step card.
- [ ] **WaitForKey — Cancel Branch Support:** Set `LastActionSucceeded := 0` when user cancels WaitForKey so Logic Blocks can handle it.
- [ ] **UserInput — Cancel Branch Support:** Set `LastActionSucceeded := 0` on InputBox cancel so Logic Block false branch handles it.
- [ ] **MouseTrace — Success/Fail Tracking:** Set `LastActionSucceeded := 0` if trace file is missing, `1` on successful playback.
- [ ] **Drag and Drop — Return Mouse to Origin:** Restore origin mouse coordinates after drag release in both AHK and C# engines.
- [ ] **Scope Variable Collision for Nested WIN_LIVE Searches:** Use step-name-prefixed variables for scope coordinates inside nested loops/logic blocks.
- [ ] **Popup Block — "On Cancel" Logic Branch:** Set `LastActionSucceeded := 0` when user clicks Cancel on Checkpoint popups.
- [ ] **CallMacro — Propagate Sub-Macro Result:** Set `LastActionSucceeded` after CallMacro based on whether sub-macro succeeded.
- [ ] **Logic Block Visual Scope Lines:** Draw VS Code-style scope brackets on the left of the timeline connecting Logic Blocks to child steps.
- [ ] **"Test This Branch" Button on Logic Block:** Add a play button on each branch of a Logic Block to test that branch in isolation.
- [ ] **Smart Retry with Backoff:** Implement stepped delays (100ms, 200ms, 400ms) for Image/Pixel search retries.
- [ ] **Per-Step Playback Speed Override:** Allow individual steps to override global macro playback speed.
- [ ] **Popup Block — PopupMode Compiler Support:** Implement distinct AHK output for Auto-Timeout and Floating Alert popup modes.
- [ ] **Keyboard Block — IsHumanized Support:** Add randomized delay before `Send()` when `IsHumanized` is checked.
- [ ] **Mouse Multi/Timer Click — Button Type Bug:** Fix button argument handling for Multiple Clicks and Timer Click modes.
- [ ] **SetVariable — IsValid Validation:** Add `!string.IsNullOrWhiteSpace(InputVariableName)` validation check.
- [ ] **UserInput Preview — Wrong Variable Name:** Fix preview execution to store result into `InputVariableName` instead of `StepName`.
- [ ] **WaitUntil — Missing from Preview Execution:** Add full polling loop for `WaitUntil` step type in `MacroExecutionService.cs`.
- [ ] **Mouse "Active Window" Target — Missing from Preview:** Add `GetForegroundWindow()` + center calculation for Active Window targets in preview.
- [ ] **Sandbox — LoopSequence and LogicIf Support:** Add `CompileSandboxStep` cases for LoopSequence and LogicIf.
- [ ] **FailIfMissing Inside Loop — Use Break:** Use `break` instead of `return` when `FailIfMissing` is triggered inside a loop.
- [ ] **AutoLaunchPath Missing from Clone:** Include `AutoLaunchPath` property in `Clone()` method for all block models.
- [ ] **SetVariable — Incomplete AHK Escaping:** Use `EscapeStringForAhkLiteral()` for SetVariable values.
- [ ] **Keyboard Combo Brace Escaping:** Escape `{` and `}` characters in `ConvertComboToAhk()`.
- [ ] **Mouse "Release Up" String Check:** Handle both `"Released Up"` and `"Release Up"` strings in Mouse block compiler.
- [ ] **Popup/Notification Message Escaping:** Use `EscapeStringForAhkLiteral()` for popup and notification messages.
- [ ] **ImageSearch FoundX/Y Reset on Failure:** Reset `FoundX := 0, FoundY := 0` at the start of ImageSearch blocks in compiled AHK.
- [ ] **UIElement SameApp Mode Main Compiler Support:** Implement `UIFindMode == "SameApp"` support in main compiled AHK output.
- [ ] **Notification Preview Behavior:** Fix preview mode to show tray notification instead of blocking modal dialog.

---

## 🗑️ Section 4: Dead Code Cleanup

---

## 🚀 Section 5: Recent Session Backlog & Design Ideas

- [ ] **Easter Egg Master Guide Hints:** Test 3 hint options (Easy, Medium, Hard) and pick 1 final level for the Master Guide.
- [ ] **Mobile Wi-Fi Remote Sidebar Section:** Elevate Mobile Remote to a dedicated left sidebar section with ON/OFF switch, QR code, and IP/PIN display (UI mockup first).
- [ ] **Advanced Engine Settings Group:** Group Turbo Engine Mode, Admin VIP Mode, and Performance Mode into one unified "Advanced Engine & Performance" card in Settings.
- [ ] **User Input Block UI Refinements:** Move question prompt into gear menu (`⚙`) and add dim placeholder pre-text (*"Type prompt here..."*) in the input text box.
  - **Core concept**: When UIElement capture starts, lock the real Windows cursor in place (`SetCursorPos` + `ClipCursor` each frame). Hide the real cursor. Render a fake "ghost" cursor on the overlay that moves freely using Raw Input (`WM_INPUT` deltas).
  - UIAutomation queries use the **ghost cursor's virtual coordinates** — the target app never receives mouse movement, so zero hover effects, zero popups, zero flickering menus.
  - Always queries **live UIAutomation tree** (no stale screenshot cache, no giant bitmap).
  - Works cleanly across **multi-monitor / high-DPI** setups.
  - **Modifier keys**:
    - `Ctrl` → freeze highlight box (ghost cursor still moves freely)
    - `Alt` → pause ghost cursor movement (everything visually freezes, no Win32 hook issues)
    - `Shift` → cycle through parent elements (Button → Toolbar → Ribbon → Window)
  - **Bonus: Magnetic Snapping** — snap to nearest valid `AutomationElement` center when controls are tiny (16×16 toolbar icons, ribbon buttons)
  - **Bonus: Smart Cache** — after ~250ms of no movement, cache bounding rect + Name + ControlType + AutomationId; refresh only when movement resumes
  - Source: `UIElementCaptureService.cs` — full rewrite of `MouseHookCallback` + `Timer_Tick` logic

- [ ] **Pixel Block Shortlisted Improvements** *(Multi-agent review session 2026-07-26)*
  - [ ] **Mini-Cluster Match (Center + 4 Neighbors)**: Record target pixel + 4 surrounding cardinal pixels (N/S/E/W). At runtime, require center + 3/4 neighbors to match before accepting. Eliminates random single-pixel false matches while remaining super fast. *(Future implementation)*
  - [ ] **Multi-Color Variant List**: Allow adding 2–3 alternative hex colors inside a single Pixel Block (e.g. normal, hover, dark mode) instead of creating multiple duplicate blocks.
  - [ ] **HSL / Hue-Lock Tolerance**: Upgrade color tolerance to lock Hue tightly (e.g. ±5°) while allowing broader Lightness variation (±60°). *(Note: Audit/check current RGB tolerance values first before implementing)*.
  - [ ] **Window-Relative Bounds Fallback**: Restrict search scope strictly to the target window bounds if the pixel was captured on a specific window.
  - [ ] **Live Tolerance Heatmap Overlay**: Live cyan overlay in eyedropper color picker showing matching pixels. *(Note: Has user doubts — evaluate further before building)*.

- [ ] **Window Block Shortlisted Improvements** *(Multi-agent review session 2026-07-26)*
  - [ ] **Process-First Matching + Fuzzy Title Fallback**: Match executable (`ahk_exe`) first. If exact title fails (e.g. document renamed `Report_v1` → `Report_v2`), match any window of that process with closest title similarity instead of auto-launching a new instance. *(Discuss before implementing)*.
  - [ ] **MRU (Most-Recently-Used) Window Selection**: For multi-instance apps (multiple Chrome or Notepad windows), rank and activate the window that was most recently active instead of a random first match.
  - [ ] **Smart Title Normalization (Badge Stripping)**: Automatically strip dynamic title noise before matching (e.g. unread counts `(12)`, notification badges, `*` unsaved indicators, `[Administrator]`).
  - [ ] **Macro-Scoped HWND Session Memory**: Cache target window handle (`HWND`) during a single macro execution run so steps 2..N reuse it instantly. *(Note: Needs further debate on pros/cons & user opinion before implementation)*.
  - [ ] **Expose "Wait Until Window Closes" Action**: Add `WinWaitClose` as an explicit action choice in the Window Block dropdown so macros can wait for popups/installers to close. *(Note: Needs user opinion/review before implementation)*.
  - [ ] **Dry-Run "Test Target" Preview Button**: Button in step editor that tests window targeting in real-time without stealing focus, reporting match status and latency. *(Note: Open for debate/refinement)*.

- [ ] **UIElement Block Shortlisted Improvements** *(Multi-agent review session 2026-07-26)*
  - [ ] **Ranked Multi-Condition Cascade (Exact → Contains → Relaxed)**: Try exact match (`AutomationId` + `Name` + `ControlType`) → `Name Contains` → `ControlType` + `ClassName`. Handles dynamic text (e.g. `Submit (3)`) and unstable IDs. *(Note: Debatable — discuss & clarify before implementing)*.
  - [ ] **RuntimeId Fast-Path Cache**: Save element `RuntimeId` in memory during macro run for instant O(1) lookup on repeated interactions. *(Note: Explain & discuss before implementing)*.
  - [ ] **Element State Guard (Enabled & Visible Check)**: Verify `IsEnabled` and `IsOffscreen` before acting. Wait for disabled elements or auto-scroll offscreen elements into view. *(Note: Explain & discuss before implementing)*.
  - [ ] **Dynamic Window-Relative Coordinate Fallback**: Calculate fallback coordinates relative to parent window's top-left corner so window moves don't click wrong spots. *(Note: Explain & discuss before implementing)*.
  - [ ] **New UI Actions (Hover, Get Bounding Rect, State Queries)**: Add `Hover`, `Get Bounding Rect` (variables), and state queries (`Is Checked`, `Is Enabled`, `Is Visible`) to action dropdown. *(Note: Explain & discuss before implementing)*.
  - [ ] **Parent Container Subtree Scoping**: Save parent container (e.g. `FormPanel`) during recording and search only inside its subtree for 10x faster execution. *(Note: Explain & discuss before implementing)*.

- [ ] **Dynamic Runtime Step Repair ("Fix It Now" Interactive Popup)** *(User Idea — 2026-07-26)*
  - **Concept**: When a macro step fails at runtime (e.g., Image, Pixel, or UIElement not found despite being visible), instead of silently aborting or throwing a raw error, display a sleek interactive popup toast: `[Fix It Now] [Skip] [Abort]`.
  - Clicking **"Fix It Now"** immediately triggers the quick capture tool over the live screen state.
  - The user re-captures/re-selects the target image/pixel/element on the spot.
  - The new target data (PNG file, hex color, or UIA properties) is dynamically updated in the database on the fly, and the macro immediately resumes execution from that step!
  - Applies to: Image Block, Pixel Block, UIElement Block, Window Block.
  - *(Note: Create a multi-agent brainstorming prompt to gather ideas & suggestions from other agents to refine and perfect this runtime repair workflow!)*

- [ ] **Simplify Macro Card Icon Picker (Preset Icons Only)** *(User Idea — 2026-07-26)*
  - Remove custom emoji input / "Enter any emoji" section and keyboard icon input from the macro card icon picker dialog.
  - Simplify the icon picker to only offer preset icon choices.

- [ ] **System Tray Context Menu UI & Layout Fixes** *(User Idea — 2026-07-26)*
  - Fix menu item text clipping/truncation (e.g. `Open Wind...` getting cut off). Set proper auto-width / padding.
  - Clean up raw hotkey formatting in menu items (e.g. `{1}` showing raw curly braces instead of clean hotkey display).

- [ ] **Remove "Show Insert Feedback" Feature & Toggle** *(User Idea — 2026-07-26)*
  - Completely remove the "Show Insert Feedback" setting toggle and its underlying visual highlight effect when adding steps/items.

- [ ] **Bug: App Switcher Fails to Activate Window Unless "Lock to This Exact Window" is Enabled** *(Bug Report — 2026-07-26)*
  - When capturing an app for App Switcher / Dashboard quick actions, triggering it fails to activate any open window of that app unless "Lock to This Exact Window" is explicitly checked.
  - Default process/app window matching/activation fails when un-locked.


- [ ] **Easter Egg Window Redesign (Maaz Secret Keyword):** Redesign the secret Easter Egg popup window that appears when saving a macro named " maaz\. Make it sleek and modern.

- [ ] **Pixel Search Target Coordinates Sync Audit Fix:** Ensure `DefaultTargetSize` setter syncs `SearchWidth`/`SearchHeight` properly during dynamic execution in `MacroExecutionService.cs`.
- [ ] **UIElement "Wait Until Gone" Polling Interval & Timeout:** Make the hardcoded 100ms polling rate configurable using dynamic `CheckIntervalMs` settings (50ms, 100ms, 250ms) and expose configurable max timeout `UITimeoutSeconds`. Added speed presets (Fast, Normal, Eco).

- [ ] **Smart System & Event Triggers** *(User Idea — 2026-07-29)*
  - **PC Startup / Windows Boot Trigger**: Automatically run macros when Windows starts up.
  - **Startup Offset / Boot Delay**: Configurable delay (e.g. wait 30s or 2m after boot) so background apps load smoothly before running.
  - **File Opened / Created Trigger**: Automatically run macros when a specific file or folder item is opened or created.
  - **App Launch / Focus Trigger**: Automatically run macros whenever a specific target app opens or comes into focus.

- [ ] **Move Macro to Profile Confirmation & Toast Notification** *(User Idea — 2026-07-31)*
  - **Confirmation Prompt**: Show a confirmation dialog (*"Move '{MacroName}' to '{ProfileName}'?"*) before moving a macro card to another profile.
  - **Controlled by Safety Confirmation Setting**: Respects the global **Safety Confirmation** setting — if turned OFF, the confirmation prompt is bypassed automatically.
  - **Toast Notification**: Always display a brief visual toast notification after moving (e.g. *"Moved to {ProfileName}"*) with an **Undo** button.

- [ ] **Inward Boundary Shimmer Effect** *(User Idea — 2026-08-01)*
  - Sleek, non-distracting inward border shimmer animation around the editor frame while live building a macro.

- [ ] **UIElement Capture — DPI Correction for BoundingRectangle** *(Audit finding — 2026-08-02)*
  - `UIElementCaptureService.PositionHighlight` places the highlight box in WPF DIPs but uses `BoundingRectangle` (physical pixels) with NO DPI division — unlike the other 3 overlays which DPI-correct. On a second monitor with different scaling, the teal/green capture box will be offset/mis-sized.
  - Fix later; requires behavior change so deliberately left out of the `NativeWindowHelper` consolidation.
  - Related: `NativeWindowHelper.GetDpiScaleAtPoint(x, y)` is available for the fix.



