---
status: fixed
priority: high
created: 2026-07-16
fixed: 2026-08-01
project: PowerX Keys
affects: Image Block Preview (right-click → Preview)
---

# Image Preview Coordinate Bug

## ✅ RESOLVED — Both fixes shipped (verified 2026-08-01)

Both encoding sites now subtract the window position before saving, so the preview searches the correct relative area:

1. **CaptureOverlay.xaml.cs:1267-1284** (drag mode) — now gets the window rect from `rootHwnd` and saves `relX1 = physX - wr.Left`, `relY1 = physY - wr.Top`, etc. (matches the proven sticky-confirm pattern; also stores `CapturedWindowX/Y`).
2. **MacroEditorViewModel.Capture.cs:341-352** (post-Image-Studio crop) — now subtracts `step.SourceWindowX/Y` when both have values: `rx1 = adjustedRefX - step.SourceWindowX.Value`, etc.

The two previously-correct paths (sticky first-pin, sticky confirm) are unchanged.

## What's Happening

When you preview an Image Block, the Smart Box shows up in the right area visually, but the image search fails to find the image. Even manually rescoping doesn't help. The "Screen Fallback" is what actually finds it (when enabled).

## Why It's Happening

The app saves the captured area as **absolute screen coordinates** (e.g., 1200, 600), but the preview engine reads them as **relative to the window** and adds them to the current window position. So it searches at completely wrong coordinates.

Example from log:
- Saved coords: `CapturedAt=(-32000,-32000)` (window was minimized during save!)
- Smart Box calculated: `(33158, 32827)` — way off-screen
- Only found image because full screen fallback kicked in

## Two Places That Save It Wrong

### 1. CaptureOverlay.xaml.cs — Line 1116 (Drag mode)

Uses absolute `physX, physY` directly:
```csharp
CapturedScope = $"WIN_SMART:{procName}:TITLE={winTitle}:{physX},{physY},{physX + physWidth},{physY + physHeight}";
```

**Should subtract window position** (like the sticky-confirm path at line 1670 already does correctly):
```csharp
// Get window rect for the target
GetWindowRect(rootHwnd, out RECT r);
int dx1 = physX - r.Left;
int dy1 = physY - r.Top;
int dx2 = (physX + physWidth) - r.Left;
int dy2 = (physY + physHeight) - r.Top;
CapturedScope = $"WIN_SMART:{procName}:TITLE={winTitle}:{dx1},{dy1},{dx2},{dy2}";
```

### 2. MacroEditorViewModel.Capture.cs — Line 297 (After Image Studio crop)

Uses absolute `adjustedRefX, adjustedRefY`:
```csharp
step.SearchScopeSummary = $"WIN_SMART:{procPart}{titlePart}:{adjustedRefX},{adjustedRefY},{adjustedRefX + adjustedRefW},{adjustedRefY + adjustedRefH}";
```

**Should subtract SourceWindowX/Y** (which is already available at this point):
```csharp
int winX = step.SourceWindowX ?? 0;
int winY = step.SourceWindowY ?? 0;
int relX1 = adjustedRefX - winX;
int relY1 = adjustedRefY - winY;
int relX2 = (adjustedRefX + adjustedRefW) - winX;
int relY2 = (adjustedRefY + adjustedRefH) - winY;
step.SearchScopeSummary = $"WIN_SMART:{procPart}{titlePart}:{relX1},{relY1},{relX2},{relY2}";
```

## Why Line 946 and Line 1674 Are Already Correct

- **Line 946** (sticky mode first-pin): saves `0,0,width,height` — full window, relative. Correct.
- **Line 1674** (sticky confirm with scope): subtracts `wRect.Left/Top`. Correct.

Only the drag-mode path (line 1116) and the post-crop rebuild (line 297) are wrong.

## Why The Sandbox Works

The sandbox (`C:\Users\Maaz\Documents\ImageSearchLab\MainWindow.xaml.cs`) doesn't use `WIN_SMART:` at all. It stores:
- Absolute capture coords (`_captureX, _captureY`) in logical pixels
- Source window position (`_sourceWinX, _sourceWinY`) queried fresh at search time via `UpdateSourceWindowBounds()`
- Smart Box = absolute `captureX - padding` → `captureX + captureW + padding`
- Window fallback = absolute `sourceWinX` → `sourceWinX + sourceWinW`

Key difference: The sandbox passes **absolute coords directly** to the AHK script. No relative encoding/decoding step. The main app encodes relative offsets into `WIN_SMART:` format, then decodes them at compile time — and the encoding is where the bug lives.

## Fix Approach (Option 2 — minimal, keeps current architecture)

Two small edits:

1. **CaptureOverlay.xaml.cs ~line 1116** — Get the window rect from `rootHwnd` and subtract before saving
2. **MacroEditorViewModel.Capture.cs ~line 297** — Subtract `step.SourceWindowX/Y` before saving

## Risk

- Low. Both changes are simple subtraction.
- The sticky-confirm path already does this correctly, so the pattern is proven.
- No architecture changes needed.

## Verification After Fix

1. Capture an image in the macro editor
2. Right-click → Preview
3. Should highlight the found image with purple border (not just show the cyan Smart Box)
4. Move the target window and preview again — should still find it
5. Manually rescope and preview — should work too
