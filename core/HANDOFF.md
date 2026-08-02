---
purpose: Session Handoff — Active context for the next agent
project: PowerX Keys
date: 2026-07-18
---

# HANDOFF.md

---

## How Maaz Works
- Short, simple, friendly — no jargon
- Ask before adding extras
- Don't run build/run unless asked (or told to auto-build at end)
- Read files before editing
- After fixing, just confirm — no recap unless asked

---

## Current Work: Capture Overlay Overhaul (PENDING — Antigravity)

### What's Been Decided
Maaz likes the Exp capture style (blue box, arrow cursor, no magnifier for image capture). It should become the **permanent default** — not a toggle. The toggle checkbox and `UseSimpleCapture` setting should be removed entirely.

### What Needs to Be Done (Full direction already given to Antigravity)

1. **Make Exp style the only style:**
   - Arrow cursor (no cross/plus) everywhere
   - Sticky box: solid blue `#3B82F6`, 2px thickness
   - Drag box: solid blue, no marching ants, no black underlay
   - Pinned: yellow `#FBBF24`
   - Remove all `UseExpStyle` / `useExpStyle` branching — just hardcode Exp style
   - Keep the guide popup (useful for shortcuts)

2. **Remove the toggle completely:**
   - `Models/AppConfig.cs` — delete `UseSimpleCapture`
   - `ViewModels/MacroEditorViewModel.Properties.cs` — delete the property
   - `ViewModels/MacroEditorViewModel.Capture.cs` — remove early-exit block, remove `expStyle` variable, remove `useExpStyle:` parameter from all overlay calls
   - `Views/Templates/SearchTemplates.xaml` — remove checkbox from both gear menus (2 places)

3. **Hide magnifier during image capture:**
   - Magnifier only for pixel/hybrid modes
   - In constructor or Loaded: `if (!CapturePixelOnly && !IsPixelModeScope && !HybridPixelMode) MagnifierBorder.Visibility = Collapsed`
   - Skip magnifier update logic in MouseMove if hidden

4. **Ctrl+Hold = hide sticky box:**
   - While holding Ctrl, hide TrackingRectangle so user can see beneath
   - Works in both floating and pinned states
   - Release Ctrl → box reappears
   - Add to guide popup: `("Ctrl Hold", "Peek beneath capture box")`

### Files to Modify
| File | What |
|------|------|
| `Views/CaptureOverlay.xaml.cs` | Main changes (style, magnifier, Ctrl feature) |
| `Views/Templates/SearchTemplates.xaml` | Remove checkbox (2 places) |
| `Models/AppConfig.cs` | Remove `UseSimpleCapture` |
| `ViewModels/MacroEditorViewModel.Properties.cs` | Remove property |
| `ViewModels/MacroEditorViewModel.Capture.cs` | Remove toggle logic + `useExpStyle` param |

---

## Also In Progress: Bug Fixes (Antigravity)

### WIN_SMART Compiler Bug
**Problem:** Main compiler (`ScriptCompilerService.cs` ~line 2068) doesn't handle `WIN_SMART:` scopes — only `"Smart Search"` and `"SMART_BOX"`. Falls through to static coords.
**Fix:** Add `WIN_SMART:` to the condition, same as already done in `SingleStep.cs`.

### SourceWindowX/Y Null on Recapture
**Problem:** Line ~218 in `MacroEditorViewModel.Capture.cs` unconditionally overwrites `step.SourceWindowX/Y` with `overlay.CapturedWindowX/Y` — which can be `null` on recapture.
**Fix:** Only overwrite if overlay returned non-null:
```csharp
if (overlay.CapturedWindowX.HasValue && overlay.CapturedWindowY.HasValue)
{
    step.SourceWindowX = overlay.CapturedWindowX;
    step.SourceWindowY = overlay.CapturedWindowY;
}
```

### Multi-Window Source Detection
**Problem:** `Process.GetProcessesByName()` grabs the first match, not the actual window clicked on.
**Fix:** Use the handle from `WindowFromPoint` (already available in overlay) to get the rect directly.

---

## Project Context
> See [[agent-onboarding]] for project layout, [[overview]] for architecture, and [[dual-execution-model]] for AHK vs C# execution.

---

## Where Detailed Work Lives
| Topic | File |
|-------|------|
| Image search audit | `wiki/bugs/exp-block-studio-audit.md` |
| Image recognition docs | `wiki/features/image-recognition.md` |
| Pinpoint capture history | `wiki/features/pinpoint-capture-investigation.md` |
| Error handling plan | `wiki/features/error-handling-cleanup.md` |
| Smart Mode edge cases | `wiki/features/smart-mode-audit.md` |

---

## Important Notes for Next Agent
- The `UseSimpleCapture` setting and `UseExpStyle` code are still in the codebase — they need to be REMOVED (not kept as dead code)
- The Exp style changes I (Kiro) made are partially applied — the other agent needs to clean them up and make them permanent
- Ctrl is currently used for magnifier zoom (Ctrl+scroll) but since magnifier will be hidden for image capture, there's no conflict with Ctrl+hold to hide box
- `ConfigureMagnifierStyle()` in CaptureOverlay currently overrides colors — when making Exp permanent, this method should only apply magnifier settings for pixel modes
