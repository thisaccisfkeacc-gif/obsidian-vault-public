# Smart Image Capture Size Limit — Final Design

> **Status:** Approved by User ✅  
> **Priority:** Enhancement (UX Simplification)  
> **Category:** Image Capture  
> **Replaces:** `EnableCaptureSizeLock` toggle in Settings

---

## Problem

Currently there's a toggle in Settings called **"Image Capture Safety Lock"** that hard-caps the selection rectangle at 400x400px during image capture.

Issues with the current approach:
1. **Confusing toggle** — users don't understand what "Safety Lock" means
2. **Frustrating UX** — the selection rectangle hits an invisible wall at 400px, feels broken
3. **Unnecessary setting** — no user should ever need to disable this protection
4. **All-or-nothing** — either hard-blocked or completely unprotected

> [!IMPORTANT]
> **Core Philosophy:** This is NOT a screenshot tool — it's an **image sample** capture. Users are taking a small reference sample for pattern matching, not a full screenshot. The system should guide them towards small, focused captures.

---

## Approved Solution: Studio Gate

Instead of blocking OR auto-downscaling, **let users capture freely** but use Image Studio as a quality gate.

### User Flow

```
1. User selects any region freely (no wall, no hard cap)
   │
2. Orange warning border appears if selection > 300px
   │
3. Selection complete → check size
   │
   ├─ ≤ 400×400 → Save normally ✅
   │
   └─ > 400×400 → Auto-open Image Studio 🔧
                    │
                    ├─ Constant warning banner:
                    │  "⚠ Selection is too large for reliable pattern matching.
                    │   Crop to a smaller, focused area for best performance."
                    │
                    ├─ Save button DISABLED until cropped ≤ 400×400
                    │
                    └─ User crops to the important part → Save enabled ✅
```

### Why This Is The Best Approach

| Approach | Problem |
|---|---|
| ❌ Hard wall (current) | Frustrating, feels broken |
| ❌ Auto-downscale | Scale mismatch, pattern matching may fail |
| ✅ **Studio Gate** | User picks the important part, exact pixels, no scaling issues |

**Key advantages:**
- No scale invariance problems — final image is exactly what the user cropped
- Teaches users the right behavior ("this is a sample, not a screenshot")
- User stays in full control of what anchor point they want
- Image Studio already exists — we just add a gate condition

---

## Implementation Details

### Step 1: Remove Hard Wall from CaptureOverlay

**File:** `CaptureOverlay.xaml.cs` (lines ~663-671)

```csharp
// REMOVE this block:
// bool hardLock = Services.ConfigManager.Current.Settings.EnableCaptureSizeLock && isPixelContext;
// if (hardLock) { if (w > maxW) w = maxW; if (h > maxH) h = maxH; }

// KEEP the orange warning border (adjust threshold to 300px):
if (w > 300 || h > 300) {
    SelectionRectangle.Stroke = warningBrush; // Orange
}
```

### Step 2: After Capture — Check Size & Route

**File:** `CaptureOverlay.xaml.cs` (capture complete handler)

```csharp
// After bitmap is captured:
if (bitmap.Width > 400 || bitmap.Height > 400)
{
    // Open Image Studio with the oversized image
    // Pass a flag: "requiresCrop = true"
    OpenImageStudio(bitmap, requiresCrop: true);
}
else
{
    // Normal flow — save directly
    SaveCapture(bitmap);
}
```

### Step 3: Image Studio — Add Crop Gate

**File:** Image Studio view/viewmodel

```csharp
// When requiresCrop is true:
// 1. Show persistent warning banner at top
// 2. Disable Save button
// 3. On crop change → check if cropped region ≤ 400×400
// 4. If yes → enable Save button
// 5. If no → keep disabled with message
```

### Step 4: Remove Toggle from Settings

- **`SettingsDashboardView.xaml`** — Collapse the "Image Capture Safety Lock" toggle
- **`SettingsView.xaml`** — Collapse the toggle from legacy view
- **`AppConfig.cs`** — Keep property for backward compat, add `[JsonIgnore]`

---

## Edge Cases

| Case | What Happens |
|------|-------------|
| User selects 50×50 (small icon) | Normal flow, saves directly |
| User selects 400×400 exactly | Normal flow, at threshold |
| User selects 600×400 | Opens Studio → must crop width to ≤ 400 |
| User selects 1920×1080 (full screen) | Opens Studio → must crop to a focused area |
| User cancels Studio without cropping | Capture is discarded (no save) |

---

## Files to Modify

| File | Change |
|------|--------|
| `CaptureOverlay.xaml.cs` | Remove hard wall, add Studio redirect |
| `ImageStudio` (view + viewmodel) | Add crop gate + warning banner |
| `SettingsDashboardView.xaml` | Collapse Safety Lock toggle |
| `SettingsView.xaml` | Collapse Safety Lock toggle |
| `AppConfig.cs` | `[JsonIgnore]` on `EnableCaptureSizeLock` |

---

## Agent Review

**Reviewer:** Antigravity (Gemini) — 2026-06-29  
**Verdict:** ✅ Approved — Studio Gate is the right approach

- The flow is clean: capture freely → Studio gate → crop → save
- No scale invariance issues (user crops exact pixels, no resizing)
- Teaches users the right mental model ("sample, not screenshot")
- Orange warning at 300px is the right trigger point
- Keep `EnableCaptureSizeLock` in config for backward compat, just ignore it
- Consider a subtle toast for small captures too: `"✅ Perfect size for matching!"`
