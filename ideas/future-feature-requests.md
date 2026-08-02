---
tags: [wiki, ideas, future, features]
date: 2026-06-24
status: active
---

# 💡 Future Feature Requests

> Low-priority ideas discussed with the user. Not urgent, but worth adding someday.
> Add new entries as they come up in conversation.

---

## 📋 Feature List

| # | Feature | Area | Effort | Priority | Date Added |
|---|---|---|---|---|---|
| 1 | Relative Pixel Mode | Pixel Search | 🟢 Easy | 🔵 Low | 2026-06-24 |
| 2 | Smart Wait Block | Macro Steps | 🟡 Medium | 🔵 Low | 2026-06-29 |

---

## 🔵 #1 — Relative Pixel Mode (PixelSearch)

**What:** When a user captures a pixel, instead of saving the absolute screen coordinate, save it as a **relative offset from the window's top-left corner**.

**How it works at runtime:**
1. Find the target window using `WinGetPos()` (like WIN_LIVE already does)
2. Add the saved relative offset: `screenX = WindowX + RelOffsetX`
3. Check the pixel color at that resolved coordinate

**Why it's low priority:**
- Image Search already covers this use case (capture a tiny screenshot of the area instead)
- Pixel color alone is fragile (themes, dark mode can change it)
- Niche use case — most users search by color, not by exact position

**Effort estimate:**
- 1 new model field: `UseRelativeToWindow` (bool) + `RelOffsetX/Y`
- ~15 lines of new AHK code in `ScriptCompilerService.cs`
- Small UI toggle in the Pixel Search step card

---

## 🔵 #2 — Smart Wait Block (WaitUntilEvent)

**What:** Combine the conditional event-based waiting actions (like *Wait for Key* and *Wait Until Image/Pixel/UI Element*) into a single unified "Smart Wait" block on the timeline.

**How it works:**
1. **Dropdown Selector:** The block has a dropdown menu with choices:
   - `⌨️ Key Press` (waits for Enter, Escape, etc.)
   - `🖼️ Visual/UI` (waits for an Image, Pixel, or UI element on screen)
   - `🪟 Window` (waits for a window to open, focus, or close - future expansion)
2. **Dynamic UI:** The block card dynamically changes its fields and buttons based on the dropdown choice.

**Why it's postponed:**
- The existing blocks (`WaitForKey` and `WaitUntil`) already get the job done.
- Changing them or introducing a new block takes more UI rework.
- Kept as a future refactoring concept.

---

## 💬 Notes

- 2026-06-24: Discussed with user. Decided to skip Relative Pixel Mode for now, revisit later.
- 2026-06-29: Discussed Smart Wait Block. Decided to postpone/hold and focus on Relative Window Search instead.
