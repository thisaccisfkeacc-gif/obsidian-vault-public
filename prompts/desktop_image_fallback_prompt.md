# 🔍 Deep Audit Request: Desktop Image Search Fallback Investigation

Please perform a deep structural and execution audit of **Desktop Image Search Fallback** in PowerX Keys.

---

## 🎯 Target Scope
- **Root Directory**: `c:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\`
- **Core Files**:
  - `PowerX.Services\Services\ScriptCompilerService.SingleStep.cs`
  - `PowerX.Services\Services\ScriptCompilerService.cs`
  - `PowerX.Services\Services\MacroExecutionService.cs`
  - `PowerX.Core\Models\MacroStep.cs`

---

## 📋 What to Audit & Verify

1. **Desktop Window Recognition**:
   - Inspect how `ScriptCompilerService` determines window targeting for Desktop captures (where window class is `Progman`, `WorkerW`, or `Desktop`).
   - Check if Desktop targets are incorrectly bound to window-relative coordinates rather than full-screen coordinates.

2. **Full-Screen Fallback Cascade**:
   - Trace the 4-tier search cascade (Last Known Coords → Smart Box → Window Scope → Full Screen Fallback).
   - Verify whether moved desktop icons properly trigger the **Full Screen Fallback** when they are moved outside their initial captured bounds.

3. **Output File**:
   - Please write your findings, verified code paths, and any fix suggestions directly to:
     `c:\Users\Maaz\Documents\New folder\desktop_image_fallback_audit.md`
