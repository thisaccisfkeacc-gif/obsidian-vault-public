# Image Search & Smart Box Coordinate Bug Investigation

## Problem
During preview, the image block is not found even when the image is visible on-screen and a highlight border is shown. Manually rescoping (click-and-drag) also fails.

## Cause
There is a coordinate system mismatch (absolute vs. relative coordinates) in two places:

1. **Cropped Image Save (`MacroEditorViewModel.Capture.cs`):**
   - Rebuilding `WIN_SMART` scope uses absolute screen coordinates (`adjustedRefX`, `adjustedRefY`):
     ```csharp
     step.SearchScopeSummary = $"WIN_SMART:{procPart}{titlePart}:{adjustedRefX},{adjustedRefY},{adjustedRefX + adjustedRefW},{adjustedRefY + adjustedRefH}";
     ```
   - However, AHK compiler expects relative offsets to the target window origin because `WIN_SMART` is evaluated as a relative scope (`isRel = true`).

2. **Scope Capture (`CaptureOverlay.xaml.cs`):**
   - When rescoping manually via click-and-drag (`CaptureScopeOnly = true`), the overlay saves the selected area using absolute screen coordinates (`physX`, `physY`) instead of relative offsets:
     ```csharp
     CapturedScope = $"WIN_SMART:{procName}:TITLE={winTitle}:{physX},{physY},{physX + physWidth},{physY + physHeight}";
     ```

## Solution Plan

→ Full fix plan with exact code: [[wiki/bugs/image-preview-coordinate-bug]]

1. **Fix `MacroEditorViewModel.Capture.cs` (line 297):**
   - Subtract `step.SourceWindowX/Y` from `adjustedRefX/Y` before saving to `WIN_SMART`.
2. **Fix `CaptureOverlay.xaml.cs` (line 1116):**
   - Get window rect from `rootHwnd`, subtract `r.Left/r.Top` from `physX/physY` before saving to `WIN_SMART`.

**Status:** Ready to fix. Waiting for approval.
