# Sandbox → Main App Migration Plan

**Purpose:** Port validated fixes/features from `ImageSearchLab` sandbox into the main `PowerX_Keys_V2` app.
**Last Updated:** 2026-07-16

Source: `PowerX Keys/ImageSearchLab/`
Target: `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/`

---

## Verdict: What Actually Needs Porting

After comparing both codebases, most sandbox features are **already handled differently (and often better)** in the main app. Here's the honest breakdown:

| Feature | Port? | Reason |
|---------|-------|--------|
| DPI-Unaware Screen Metrics | NO | Main app uses per-monitor `GetDpiForMonitor` — more correct than sandbox's system-wide approach |
| Logical/Physical Coord Separation | NO | Main app already does this via `TransformToDevice` matrix |
| Capture Overlay Hit-Test Fix | NO | Main app uses ImageBrush background (opaque at runtime) + `IsHitTestVisible="False"` on children — different architecture, no bug |
| Dual Capture Mode (Sticky + Drag) | MAYBE | Main app already has drag capture. Sticky box mode may not exist — verify if wanted |
| Legacy Multi-Match Loop | YES | Main app only finds 1 match with Legacy. This is genuinely new. |
| FindText Multi-Match | NO | Main app already handles this (`ok.Length` loop) |
| Auto-Detect Source Window | NO | Main app already stores `CapturedProcessName`/`CapturedWindowTitle` at capture time |
| Dynamic Window Bounds Tracking | NO | Main app uses AHK-side `WinExist`+`WinGetPos` at runtime — equivalent |
| Last Position Cache | NO | Main app already has `LastKnownX/Y/W/H` with threshold |
| 4-Step Cascade | NO | Main app already implements this in `ScriptCompilerService.SingleStep.cs` |
| AHK Timeout Reduction | MAYBE | Worth tightening if main app uses 10s+ timeouts currently |
| Window Title Parsing | MAYBE | Could improve the source window label display |
| Tolerance Dropdown Cleanup | YES | Main app's Legacy dropdown is inconsistent (11 vs 7 options between views) |

---

## Items to Port (Final List)

### 1. Legacy Multi-Match Loop (Priority: High)

**Current main app behavior:** Single `ImageSearch` call per cascade step. Finds first match only.

**What to port:** The looping AHK script pattern from sandbox:
```ahk
Loop {
    if (A_TickCount - startTime > 3000)  ; Time guard
        break
    result := ImageSearch(&FoundX, &FoundY, curX, curY, searchX2, searchY2, "*tolerance imagePath")
    if (!result) {
        if (curX > searchX1) {
            curX := searchX1
            curY := curY + 1  ; Move to next row
            if (curY >= searchY2)
                break
            continue
        }
        break
    }
    matchCount++
    allMatches .= FoundX . "," . FoundY . "," . imgW . "," . imgH . ";"
    curX := FoundX + imgW  ; Skip past found match
    if (matchCount >= 50)
        break
}
```

**Where it goes:** `ScriptCompilerService.SingleStep.cs` — the Legacy engine branch (when `UseFastEngine == false`). Only active when `step.FindAllMatches == true`.

**Decision needed:** Use the 1px-Y-fallback + 3s time guard (current sandbox, thorough) or imgH-Y-jump (faster, may miss vertically-close matches)?

**Recommendation:** 1px fallback + 3s time guard. The time guard prevents hangs regardless of step size, and for the "Find All" use case, thoroughness matters more than speed.

---

### 2. Tolerance Dropdown Consistency Fix (Priority: Medium)

**Current main app problem:**
- FindText dropdown: 3 presets only (Strict 0.01, Normal 0.025, Loose 0.05)
- Legacy dropdown: **inconsistent** — expanded popup has 11 options, inline popup has 7 options
- `FindTextBgTolerance` has NO UI at all (hardcoded to 0.0)

**What to port:** Clean, consistent preset lists for both engines.

**Proposed new presets:**

FindText (5 options, replacing current 3):
| Label | Fg Tolerance | Bg Tolerance |
|-------|-------------|-------------|
| Exact | 0.000 | 0.000 |
| Strict | 0.010 | 0.000 |
| Normal | 0.025 | 0.000 |
| Relaxed | 0.050 | 0.010 |
| Loose | 0.100 | 0.050 |

Legacy (5 options, replacing current 11/7):
| Label | *Variation |
|-------|-----------|
| Exact | 0 |
| Strict | 5 |
| Normal | 15 |
| Relaxed | 30 |
| Loose | 50 |

**Where it goes:** `Views/Templates/SearchTemplates.xaml` — both `ImagePanelTemplate` and `ImagePanelInlineTemplate` sections.

**Decision needed:** Should we expose `FindTextBgTolerance` in the UI via these presets, or keep it hidden at 0? Sandbox bundles it into the preset pair.

---

### 3. AHK Timeout Tightening (Priority: Low)

**What to do:** Check main app's current AHK process timeout. If it's 10s+, reduce to 5s and add a 3s in-script time guard for Legacy multi-match loops.

**Where:** Wherever the main app launches and waits for AHK processes.

---

## Items NOT to Port (and why)

| Item | Why skip |
|------|----------|
| DPI handling | Main app's `GetDpiForMonitor` per-monitor approach is more correct than sandbox's system-wide `GetDeviceCaps`. No regression to fix. |
| Overlay hit-test/background | Main app uses ImageBrush screenshot as background (opaque) and already sets `IsHitTestVisible="False"` on children. Different architecture, no click-through bug. |
| Window detection C#-side | Main app does this AHK-side with `WinExist`+`WinGetPos` at runtime — more reliable since it gets live bounds in the same process/thread as the search. |
| Last Position Cache | Already exists in main app with same threshold logic. |
| Cascade order | Already implemented identically in main app. |

---

## Porting Order

1. **Tolerance Dropdown Cleanup** (#2) — low risk, fixes existing inconsistency, no behavior change
2. **Legacy Multi-Match Loop** (#1) — new feature, gated behind `FindAllMatches` flag so existing single-match behavior is untouched
3. **AHK Timeout** (#3) — trivial, do alongside #1

---

## Open Questions (need your input)

1. Should the new FindText presets expose `BgTolerance` via the dropdown (bundled into each preset), or leave it at 0?
2. For Legacy multi-match: use the sandbox's 1px-Y-step + 3s guard, or the faster imgH-jump version?
3. Does the main app's sticky-box capture mode already exist? If not, is it wanted?
