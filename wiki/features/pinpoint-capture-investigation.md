---
tags: [feature, research, window-capture, pinpoint]
date: 2026-07-04
status: investigation
---

# 🎯 Pinpoint Capture — Historical Restoration Plan

**Summary:** This document serves as the absolute "Source of Truth" for the Pinpoint window capture feature. It documents exactly how it was built before recent modifications.

---

## 🕵️‍♂️ The Original "Pinpoint" (FACTUAL)

Based on a deep dive into session `96b4d703-0527-4e0a-b626-a0e5f3d4298b` (Step 942), here is the original blueprint:

### 1. The Core UI Trigger
- **File:** [MacroEditorViewModel.Commands.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/ViewModels/MacroEditorViewModel.Commands.cs)
- **Original Logic:** It did NOT call a background service. It opened the `CaptureOverlay` window directly:
  ```csharp
  CaptureOverlay overlay = new CaptureOverlay(captureWindowOnly: true);
  if (overlay.ShowDialog() == true) { ... }
  ```

### 2. The Visual Highlight (Window Edges)
- **File:** [CaptureOverlay.xaml.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/CaptureOverlay.xaml.cs)
- **Logic:** It used a dedicated `_smartHighlightRect` (XAML element) inside the overlay.
- **Snapping:** It used a `while` loop with `GetWindow(currHwnd, GW_HWNDNEXT)` and `GetAncestor(currHwnd, GA_ROOT)` to find the precise window frame under the mouse.
- **Fullscreen Protection:** It included logic to *hide* the highlight if the window was fullscreen/maximized to prevent "blocking" the screen view.

---

## 🏛️ Why it "Broke"
On July 3rd, the code was "refactored" to move logic into `WindowCaptureService`. This introduced:
- A new, non-XAML `Rectangle` created in code.
- A different DPI scaling method.
- A simplified search loop that lost the "edge-perfect" snapping of the original `CaptureOverlay`.

---

## 🔍 Code Verification (July 4th Audit)

> [!IMPORTANT]
> Full code verification performed against the actual current source. Every claim checked line-by-line.

### ⚡ Critical Discovery: Two Separate Commands Exist

The codebase has **two different** capture commands, NOT one:

| Command | Where (line) | What it calls | UI Type |
|---|---|---|---|
| `CaptureWindowPinpointCommand` | Line 766 | `WindowCaptureService.CaptureInteractiveWindowAsync()` | Transparent picker (no screenshot) |
| `CaptureInteractiveWindowCommand` | Line 831 | `CaptureOverlay(captureWindowOnly: true)` | Full frozen screenshot overlay |

So the original document's claim about `CaptureOverlay(captureWindowOnly: true)` is **correct** — that code still exists, just on the *other* command (`CaptureInteractiveWindowCommand`). The "Pinpoint" command (`CaptureWindowPinpointCommand`) was the one refactored to use `WindowCaptureService`.

---

### Claim 1: `CaptureOverlay(captureWindowOnly: true)` constructor
- **Status:** ✅ **Confirmed — EXISTS**
- Constructor at line 141 of CaptureOverlay.xaml.cs:
  ```csharp
  public CaptureOverlay(bool captureScopeOnly = false, bool capturePixelOnly = false,
      bool captureWindowOnly = false, int targetWidth = 30, int targetHeight = 30, ...)
  ```
- When `CaptureWindowOnly` is true, it shows guide text (line 217):
  ```csharp
  GuideTitle.Text = "🪟 PINPOINT WINDOW";
  GuideDesc.Text = "Hover over any window and single-click to capture it.";
  ```

### Claim 2: `_smartHighlightRect` existed
- **Status:** ✅ **CONFIRMED — EXISTS for BOTH CaptureScopeOnly AND CaptureWindowOnly**
- Declared at line 976, created in `Window_Loaded` at lines 1097-1108
- **Condition at line 1093:** `if (CaptureScopeOnly || CaptureWindowOnly)` — both modes create the highlight rectangle.
- ~~**Key issue:** It's only created when `CaptureScopeOnly == true`, NOT when `CaptureWindowOnly == true`.~~ **CORRECTED July 4 (2nd pass):** This was fixed — the condition now includes `CaptureWindowOnly`.

### Claim 3: Snapping via `GetWindow(GW_HWNDNEXT)` + `GetAncestor(GA_ROOT)`
- **Status:** ✅ **Confirmed — EXISTS in CaptureOverlay**
- Found in `Window_MouseMove` at lines 551-583:
  ```csharp
  IntPtr currHwnd = GetWindow(myHwnd, GW_HWNDNEXT);
  while (currHwnd != IntPtr.Zero)
  {
      if (IsWindowVisible(currHwnd))
      {
          GetWindowRect(currHwnd, out RECT rect);
          // hit test...
          IntPtr rootWnd = GetAncestor(currHwnd, GA_ROOT);
          // filter shell/desktop...
      }
      currHwnd = GetWindow(currHwnd, GW_HWNDNEXT);
  }
  ```
- ✅ This code runs when `_smartHighlightRect != null`, which is true for both `CaptureScopeOnly` and `CaptureWindowOnly` modes (fixed July 4).

### Claim 4: Fullscreen protection
- **Status:** ✅ **Confirmed**
- At lines 590-614 in CaptureOverlay.xaml.cs:
  ```csharp
  bool isMonitorFullScreen = (Math.Abs(w - screenPhysWidth) <= 20 && ...);
  if (isMonitorFullScreen || isVirtualFullScreen)
  {
      _smartHighlightRect.Visibility = Visibility.Hidden;
  }
  ```

### Claim 5: Refactor moved logic into `WindowCaptureService`
- **Status:** ✅ **Confirmed**
- [WindowCaptureService.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Services/WindowCaptureService.cs) — 604 lines, two public methods:
  - `CaptureInteractiveWindowAsync()` — transparent picker, 60FPS tracking, limited 10-iteration snapping loop
  - `CaptureInteractiveWindowWithBoundsAsync()` — same but returns window bounds
- **Key gaps vs CaptureOverlay:** No frozen screenshot, no magnifier, no tracking boxes, limited snapping iterations

### Claim 6: `ExecuteCaptureAppFromList` in ScriptLibraryViewModel
- **Status:** ✅ **Confirmed — but opens WindowPickerWindow, not CaptureLibraryWindow**
- Found at line 650 of [ScriptLibraryViewModel.Commands.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/ViewModels/ScriptLibraryViewModel.Commands.cs):
  ```csharp
  var picker = new WindowPickerWindow();
  if (picker.ShowDialog() == true) { TryAddTargetApp(action, picker.SelectedProcessName); }
  ```

---

## 📊 Current Architecture (As of July 4th)

### Capture Commands in MacroEditorViewModel.Commands.cs

```
CaptureWindowPinpointCommand (line 766)
    → WindowCaptureService.CaptureInteractiveWindowAsync()
    → Transparent picker (no frozen screen)
    → Returns: AhkTarget, TabName, AutoPath
    → Auto-saves to WindowLibrary

CaptureInteractiveWindowCommand (line 831)
    → CaptureOverlay(captureWindowOnly: true)
    → Full frozen screenshot overlay
    → Returns: CapturedScope string
    → Auto-saves to WindowLibrary
```

### Capture Commands in ScriptLibraryViewModel.Commands.cs

```
ExecuteCaptureAppBound (line 355)
    → WindowCaptureService.CaptureInteractiveWindowWithBoundsAsync()
    → Returns: Identifier, Hwnd, X/Y/W/H

ExecuteCaptureApp (line 592)
    → WindowCaptureService.CaptureInteractiveWindowAsync()

ExecuteCaptureAppFromList (line 650)
    → WindowPickerWindow dialog
    → Shows running processes list
```

### Library Save Flow

```
Capture (any command) → MacroDatabase.SaveToWindowLibrary(entry)
                              ↓
CaptureLibraryWindow → MacroDatabase.GetAllWindowLibraryEntries()
                              ↓
PickFromLibraryCommand → applies win.SelectedWindow to step
```

---

## ✅ Previously Reported Bug — NOW RESOLVED

> [!NOTE]
> **CORRECTED (July 4, 2nd pass):** The `_smartHighlightRect` IS created for `CaptureWindowOnly` mode. Line 1093 of CaptureOverlay.xaml.cs reads: `if (CaptureScopeOnly || CaptureWindowOnly)`. The snapping highlight works in both modes. The original claim that it was missing was based on an earlier state of the code.

---

## 🛠️ Restoration Roadmap (STRICT)

> [!CAUTION]
> DO NOT invent new classes. Follow the original blueprint below.

### Step 1: Revert Command Logic
- [ ] Go to `MacroEditorViewModel.Commands.cs`.
- [ ] Change `CaptureWindowPinpointCommand` back to using `new CaptureOverlay(captureWindowOnly: true)`.

> [!NOTE]
> The `captureWindowOnly` parameter **already exists** on CaptureOverlay's constructor (verified at line 141). No need to re-add it.

### Step 2: Fix `_smartHighlightRect` for CaptureWindowOnly
- [x] ~~In `CaptureOverlay.Window_Loaded`, the `_smartHighlightRect` is only created when `CaptureScopeOnly == true`.~~ **ALREADY DONE** — Line 1093 now reads `if (CaptureScopeOnly || CaptureWindowOnly)`, so the highlight rect is created for both modes.

### Step 3: Verify Library Sync
- [ ] Investigating why `ExecuteCaptureAppFromList` (in `ScriptLibraryViewModel.Commands.cs`) might fail to show the app in the list immediately after selection.

---

## 🔗 Source Files
- [CaptureOverlay.xaml.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/CaptureOverlay.xaml.cs) — Contains the "Smart Snap" logic + `captureWindowOnly` mode (1520 lines)
- [MacroEditorViewModel.Commands.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/ViewModels/MacroEditorViewModel.Commands.cs) — Two capture commands: Pinpoint (line 766) + Interactive (line 831)
- [WindowCaptureService.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Services/WindowCaptureService.cs) — The refactored lightweight replacement (604 lines)
- [ScriptLibraryViewModel.Commands.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/ViewModels/ScriptLibraryViewModel.Commands.cs) — Library selection + app-bound capture flows

---
*Verified by: Antigravity Archeology Team*
*Code audit performed: July 4, 2026 — Deep verification pass*
*2nd pass correction: July 4, 2026 — Fixed outdated Claim 2, Real Bug section, and Step 2 roadmap status. Reason: `_smartHighlightRect` creation condition at line 1093 already includes `CaptureWindowOnly`.*
