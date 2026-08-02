# Premium Light Mode Design Guidelines (UI/UX Best Practices)

A design reference manual for constructing beautiful, accessible, and high-performance light modes.

---

## 1. Contrast & Accessibility Standards (WCAG 2.2 Compliance)
To maintain accessibility, every element must achieve strict visual separation:
*   **Body & Primary Text:** Minimum **4.5:1** contrast ratio. Avoid pure black `#000000` (which causes eye glare on bright displays); prefer dark charcoal or slate (e.g. `#18181F` or `#1E202B`).
*   **Muted Text / Secondary Labels:** Minimum **3:1** contrast ratio. Use cool slate-grey (e.g. `#626674`) to separate metadata from headers.
*   **Interactive Components (Icons, Borders, Focus):** Minimum **3:1** contrast against their background.

---

## 2. Perceptual Palette Construction (No Muddy Grays)
Simple color inversion (converting dark mode values directly to light mode) makes interfaces look muddy and dead.
*   **Hue Shifting:** Instead of using pure gray scales (`#808080`), inject minor tints of blue, purple, or indigo (e.g., `#F4F5F9` instead of `#F4F4F4`). This keeps the interface feeling clean and alive.
*   **Layering (Visual Hierarchy):**
    *   *App Base (lowest layer):* Cool light-grey gradient (`#F4F5F8` -> `#EFEFF4`).
    *   *Sidebar (mid layer):* Clean off-white (`#F8F9FB`).
    *   *Cards & Active Focus (highest layer):* Pure white (`#FFFFFF`) with thin, crisp borders (`#E6E8EE`).

---

## 3. Shadows & Borders (The Subtle Depth Trick)
Premium apps (Linear, Slack) do not use heavy black dropshadows. Instead, they use a combination of a micro-border and a highly diffused shadow:
*   **Crisp Border:** 1px width with 4% to 8% black opacity (`rgba(0, 0, 0, 0.05)` or `#08000000`).
*   **Diffused Elevation Shadow:** A shadow with a large blur radius and very low opacity.
    *   *XAML Shadow Example:* `BlurRadius="8" ShadowDepth="1.5" Direction="270" Opacity="0.06"`

---

## 4. State Interactions (Hover, Active, Focused)
*   **Hover state:** A subtle 4-6% dark/purple tint over the card (`rgba(0, 0, 0, 0.04)` or `#0B000000`).
*   **Active selection:** A soft, desaturated accent wash (e.g., `#EFF1FC`) combined with a primary color accent indicator bar (like a purple vertical line).
