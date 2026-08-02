---
tags: [feature, ui, ux, design]
date: 2026-06-06
sources:
  - SmartMenuMockup/SmartMenu_WebDesign.html
status: active
---

# Smart Menu — UI/UX Design Spec 🎨

**Summary:** The unified visual and interaction design guide for the Smart Menu, ensuring consistent layouts, animations, typography, and styling tokens across all features.

---

## 🎨 Theme & Visual Tokens

The Smart Menu uses a dark, premium, glassmorphism-based design.

| Token | CSS Value | Usage |
|-------|-----------|-------|
| **Background** | `rgba(9, 9, 13, 0.75)` | Main card body (blur: 20px) |
| **Accent Primary** | `#A78BFA` | Active tabs, settings toggles, neon highlights |
| **Hover Card** | `rgba(167, 139, 250, 0.08)` | List row mouse-hover background |
| **Border Active** | `3px solid #A78BFA` | Left neon glow strip for selected rows |
| **Text Primary** | `#FFFFFF` | Main headers and focused row text |
| **Text Secondary** | `rgba(255, 255, 255, 0.5)` | Inactive row text and labels |
| **Text Muted** | `rgba(255, 255, 255, 0.18)` | Subtexts, app sources, timestamps |

---

## 📐 Layout & Components

### 1. Main Search & Navigation Header
*   **Search Input:** Large `19px` text, caret-color `#A78BFA`. Uses `Segoe UI Variable Display` or `Inter` font.
*   **Buttons:** Muted `⚙️` (settings) and `✕` (close/clear) buttons that brighten and get a dark circle hover background on mouseover.
*   **Tab Switcher:** Located directly below the search bar. Items (`All`, `Snippets`, `History`, `Files`) switch active states with a smooth transition and show a purple indicator line underneath.

### 2. History & Snippet Row Items (Visual Preview)
*   **Default Text Row:** Tighter padding, compact font size `11.5px`. Text is clipped with an ellipsis (`…`) if it exceeds the card width.
*   **Image Preview Card:** Small, clean rectangular thumbnail (`max-height: 60px; max-width: 160px;`) with a light border, displaying a relative time badge (e.g. `2m ago`) and app source on the right.
*   **Selected Row:** Highlighted with a subtle purple background overlay and a `3px` left neon border strip.

### 3. Full-Screen Settings Overlay Page
*   **Transition:** Slides up smoothly from the bottom when clicking `⚙️` settings icon.
*   **Layout:**
    *   Header: Left arrow button `←` to close, and a title `"Settings"`.
    *   Sections: Labeled in uppercase violet text (`GENERAL`, `CLIPBOARD`, `DANGER ZONE`).
    *   Controls: Right-aligned switches, selection menus, and a dynamic **Hotkey Picker button** (styled in JetBrains Mono font) that handles key combination recording on-the-fly and sends the updated hotkey string to the C# application to dynamically re-bind the keyboard hook.

---

## 🎬 Animations & Interactions

*   **App Launch Entrance:** Fade-in combined with a slight slide-up `translateY(8px)` (200ms cubic-bezier).
*   **Row Selection:** Keypress `ArrowUp` / `ArrowDown` scrolls rows into view instantly with smooth selection transitions.
*   **Pinning Indicator:** Pressing `Ctrl + Click` toggles a `📌` indicator on the right, dynamically sorting pinned items to the top without reloading the list.
*   **Setting Back Transition:** Clicking the back button `←` slides the settings overlay down and restores focus to the search bar.

---

## Key Files
- [SmartMenu_WebDesign.html](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/SmartMenuMockup/SmartMenu_WebDesign.html) — Unified frontend styles and CSS layout definitions.

## Related Pages
- [[snippets-redesign-ideas]]
- [[clipboard-manager-proposals]]
- [[index]]
