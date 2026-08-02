# Smart View — Reactivation Plan

---

## What is it? (Simple)

When you record a macro, the app captures every tiny detail — every key hold, every release, every millisecond. That's **Raw Mode**. It works but the timeline looks messy.

**Smart View** cleans it up automatically:
- Type "hello" → shows one clean **Type Text: hello** block instead of 10 separate entries
- Press Ctrl+C → shows one clean **Ctrl+C** block instead of Hold Ctrl, Press C, Release C, Release Ctrl
- Single key press → shows one **Key Press** block instead of Hold + Release

You can switch between Smart and Raw anytime. Nothing gets deleted — it's just a different way of displaying the same data.

---

## The Toggle

- Lives in the **editor toolbar** as a cycle-style button (same style as the Hide Delays button)
- **Smart Mode = ON by default**
- **Raw Mode = OFF** — shows every individual key press, useful for advanced editing
- Switching is instant, no data lost either way

---

## Phase Plan — What Gets Bundled and When

### Phase 1 — Keyboard only
- **Text bundling** — typing "hello" → one Type Text block
- **Shortcut bundling** — Ctrl+C → one block
- **Hold+Release cleanup** — single key press → one block *(except if something is between the hold and release)*

### Phase 2 — Mouse (limited)
- **Multiple clicks bundling** — double click, triple click, etc.
- **Scroll wheel bundling** — 3 scrolls → one Scroll ×3 block

### Permanently untouched — never implement
- Mouse drag / hold+release mouse
- Mouse path blocks
- Any mouse movement bundling

---

## What We're Doing — Step by Step

### Step 1 — Restore the file
The full Smart View engine (1900+ lines) is archived and ready. It just needs to be restored to:
`PowerX_Keys_V2/ViewModels/MacroEditorViewModel.SmartView.cs`

### Step 2 — Add the toggle button
A simple cycle button in the toolbar. Click once = Smart, click again = Raw. Saves to settings so it remembers your preference.

### Step 3 — Wire up `IsSmartMode`
The property already exists. Just needs to be connected to the button and to the display refresh logic.

### Step 4 — Test
Start with simple recordings (type text, press a key, do a shortcut) and verify the bundling looks right before enabling for everyone.

---

## What Smart View Handles (Full List)

| What you do | Raw Mode shows | Smart Mode shows |
|---|---|---|
| Type "hello" | 10 key hold/release steps | 1 Type Text block |
| Press Ctrl+C | 4 steps (hold ctrl, hold c, release c, release ctrl) | 1 Ctrl+C block |
| Press Enter once | 2 steps (hold + release) | 1 Key Press block |
| Click mouse | 2 steps (hold down + released up) | 1 Left Click block |
| Double click | 4 steps | 1 Double Click block |
| Scroll 3 times | 3 scroll steps | 1 Scroll block (×3) |
| Drag mouse | hold + trace path + release | 1 Drag and Drop block |
| Type "HELLO" with Shift | Shift hold + letters + Shift release | 1 Type Text: HELLO block |
| Type "HELLO" with Caps Lock | letters only | 1 Type Text: HELLO block |

---

## The Exception Rule (Important)

If something comes **between** a Hold and Release — like a delay or another action — Smart View leaves it as-is. It won't bundle it because that gap is intentional.

Example: Hold Ctrl → Wait 2 seconds → Release Ctrl → that stays split, not bundled.

---
---

# Technical Details (For Implementation Reference)

## File Location
- **Archived:** `Obsidian Vault/Archive/Smart_Mode_Logic_Backup.md`
- **Restore to:** `PowerX_Keys_V2/ViewModels/MacroEditorViewModel.SmartView.cs`
- **Lines:** 1903

## Key Properties & Flags

| Property | Location | Notes |
|---|---|---|
| `IsSmartMode` | `MacroEditorViewModel.Properties.cs` | Already exists, just needs UI wiring |
| `EnableSmartBundling` | `AppConfig.cs` | Already exists in settings |
| `EnableModifierWrapDetection` | `AppConfig.cs` | Already exists, was always-on |
| `IsManuallyAdded` | `MacroStep` | Manually added steps are never bundled |
| `VirtualSourceSteps` | `MacroStep` | List of raw steps that a Smart View block represents |
| `IsRegenerated` | `MacroStep` | Marks steps that were auto-generated (no AUTO badge shown) |
| `IsSynthetic` | `MacroStep` | Marks steps created during text regeneration |

## How the Engine Works

The main entry point is `RefreshDisplayStepsAsync()`. It:
1. Takes a snapshot of `CurrentMacro.MacroSteps` (the raw data)
2. Runs `PopulateDisplayStepsBackground()` on a background thread
3. Maps the result back to `MacroStep` objects and syncs to `_internalSourceSteps`
4. `DisplayMacroSteps` (the CollectionView bound to the timeline) filters from that

`PopulateDisplayStepsBackground()` has 4 bundling passes in Smart Mode:

### Pass 1 — Modifier Wrap Detection
Detects `Hold Down [modifier]` → inner keys → `Release [modifier]` pattern.
Collapses into shortcut blocks like `Ctrl+C`, `Ctrl+Shift+Z`.

Special case: **Shift + 2+ typing keys** → skips wrap, lets text bundler handle it (so typing HELLO doesn't become Shift+H+E+L+L+O).

### Pass 2 — Simple Key Press Bundler
`Hold Down [key]` → small delays only → `Released Up [key]` → collapses to single `Key Press` block.
Only applies to non-typing keys (so letters/numbers go to text bundler instead).

### Pass 3 — Visual Text Bundling
Consecutive typing keys → single `Type Text` block.
Handles Shift (uppercase), CapsLock state, Backspace (removes last char from bundle).
Skips if a non-shift modifier (Ctrl/Alt/Win) is held — that means it's a shortcut, not typing.

### Pass 4 — Mouse Bundling
- `Hold Down` + `Released Up` at same position → `Left Click` / `Right Click`
- Multiple clicks at same position → `Double Click` or `Multiple Clicks (×N)`
- `Hold Down` + mouse trace + `Released Up` at different position → `Drag and Drop`
- Consecutive scrolls → single `Scroll ×N` block

### Post-Processing Passes (after all 4)
- **Merge adjacent delays** — two delays next to each other become one
- **Absorb small delays between shortcuts** — `Ctrl+C → 400ms → Ctrl+V` becomes `Ctrl+C → Ctrl+V`
- **Absorb leading/trailing small delays around shortcuts** — cleans up orphaned timing gaps
- **MousePath + small delay + Click** → absorb delay into path
- **MousePath + Drag** → merge into single drag block

## Virtual Steps & Editing

When Smart View creates a bundled block (e.g. `Type Text: hello`), it's a **virtual step**. It doesn't exist in `CurrentMacro.MacroSteps` — the raw steps do. The virtual step holds a `VirtualSourceSteps` list pointing to the underlying raw steps.

Editing a virtual step triggers sync-back handlers:
- `HandleVirtualTextChange()` — text edited → regenerates underlying key press steps
- `HandleVirtualDelayChange()` — delay edited → updates first raw delay, removes extras
- `HandleVirtualMouseCoordChange()` — X/Y edited → updates hold step coords, release step EndX/EndY

Deleting a virtual step deletes all its `VirtualSourceSteps` from the raw collection.

## Known Issues (from Antigravity bug list)
- `M-1`: `HandleVirtualTextChange` doesn't push undo state — fixed in current archived version
- `M-5`: Filter captures stale `isDelayHiddenVal` — fixed (now read fresh each refresh)
- `M-6`: Orphaned key release lookback hardcoded to 5 steps — may miss distant holds
- `M-7`: `RegenerateKeyboardStepsFromText` doesn't add Shift for uppercase — current version handles this via `IsSynthetic` flag
- `M-10`: Timer leak on each refresh — fixed (CancellationTokenSource pattern used)
- `M-11`: Anonymous PropertyChanged handlers on virtual steps — still present, low risk

## CapsLock Handling
`GetCapsLockStateAt()` scans backward from the current step to find the nearest `IsRecordingStart` marker, reads its `InitialCapsLock` state, then replays all CapsLock toggles forward to determine current state. This is O(n) per character — could be optimized but acceptable for typical macro sizes.

## Hardcoding to Remove
In `MacroEditorViewModel.Properties.cs`, the display refresh is currently hardcoded to Raw Mode (`isSmart = false` or similar). This line needs to be updated to read `IsSmartMode` from the property.

## Toggle Button Plan
- Style: same as Hide Delays toggle (cycle button, no dropdown)
- Icon: a "sparkle" or "wand" icon (&#xE8D0; or &#xED28; from Segoe MDL2) for Smart, plain list icon for Raw
- Position: toolbar, next to Hide Delays button
- Binding: `IsChecked="{Binding IsSmartMode}"`
- Default: ON (Smart Mode)
