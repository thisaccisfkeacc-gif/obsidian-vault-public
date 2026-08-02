# Light Mode Aesthetics Research

A shared workspace document for PowerX Keys light mode color optimization.
Last updated: 2026-07-16

---

## 1. Current State Diagnostics
*   **The Issue:** The current Light Mode feels flat, basic, and lacks a premium visual hierarchy.
*   **Backgrounds:** A single solid `#F4F4F7` background color makes the application feel like a generic spreadsheet rather than a premium macro tool.
*   **Card Contrast:** Plain white cards on a grey background have a stark contrast transition, but lack depth, borders, or soft shadows to make them pop.
*   **Sidebars:** The sidebar has a solid color with plain list items that lack polished hover/active animations, looking disconnected from the main dashboard.
*   **Colors:** Pure black `#000000` and harsh grays create high but un-harmonized contrast. Modern premium UI uses charcoal, slate, and translucent borders.

---

## 2. Professional App Light Mode Reference (Research)

### A. Notion
- **Page background:** Soft warm off-white `#F7F6F3` (not pure white)
- **Default text:** Deep warm near-black `#37352F` (calligraphic feel, not mechanical)
- **Gray text:** `#787774`
- **Card/panel bg:** `#FFFFFF`
- **Subtle backgrounds (tags, badges):** `#F1F1EF` (gray), `#F3EEEE` (brown), `#F8ECDF` (orange)
- **Sidebar:** Clean white with soft gray hover states
- **Key insight:** Warm undertone throughout — never cold/blue-gray

### B. Discord (Light Theme)
- **Primary background:** `#FFFFFF`
- **Secondary background (sidebar/panels):** `#F5F5F5`
- **Tertiary background (inputs/recessed):** `#E8E8E8`
- **Primary text (headers):** `#070707`
- **Normal text:** `#383838`
- **Secondary text:** `#606060`
- **Muted text:** `#8D8D8D`
- **Channels/nav default:** `#808080`
- **Interactive hover:** `#383838`
- **Interactive active:** `#070707`
- **Hover modifier:** `rgba(141, 116, 116, 0.08)`
- **Key insight:** Pure neutral grays with no undertone, very subtle hover overlays

### C. Apple HIG (iOS/macOS Light Mode)
- **System background:** `rgb(242, 242, 247)` = `#F2F2F7`
- **Gray scale (6 levels):**
  - Gray 1: `rgb(142, 142, 147)` = `#8E8E93`
  - Gray 2: `rgb(174, 174, 178)` = `#AEAEB2`
  - Gray 3: `rgb(199, 199, 204)` = `#C7C7CC`
  - Gray 4: `rgb(209, 209, 214)` = `#D1D1D6`
  - Gray 5: `rgb(229, 229, 234)` = `#E5E5EA`
  - Gray 6: `rgb(242, 242, 247)` = `#F2F2F7`
- **Accent purple:** `rgb(175, 82, 222)` = `#AF52DE`
- **Accent blue:** `rgb(0, 122, 255)` = `#007AFF`
- **Key insight:** Cool purple-tinted gray (not warm, not pure neutral). System colors stay vibrant even in light mode.

### D. Radix UI / Slate Scale (12-step system for light mode)
This is the gold standard for semantic color layering:
- **Step 1 (App bg):** `#FBFCFD`
- **Step 2 (Subtle bg):** `#F8F9FA`
- **Step 3 (UI element bg):** `#F1F3F5`
- **Step 4 (Hovered bg):** `#ECEEF0`
- **Step 5 (Active/selected bg):** `#E6E8EB`
- **Step 6 (Subtle borders):** `#DFE3E6`
- **Step 7 (Element border):** `#D7DBDF`
- **Step 8 (Hovered border):** `#C1C8CD`
- **Step 9 (Solid bg / badge fill):** `#889096`
- **Step 10 (Hovered solid):** `#7E868C`
- **Step 11 (Low-contrast text):** `#687076`
- **Step 12 (High-contrast text):** `#11181C`
- **Key insight:** 12-step system gives proper layering. Steps 1-2 for backgrounds, 3-5 for interactive surfaces, 6-8 for borders, 9-10 for solid elements, 11-12 for text.

### E. Linear App
- **Brand accent:** Indigo-violet `#5E6AD2` (bg) / `#7170FF` (interactive)
- **Light mode approach:** Almost fully achromatic with one accent color
- **Background:** Near-white with cool gray panels
- **Key insight:** Minimal color usage, one strong accent, everything else neutral

### F. VS Code (Default Light+ Theme)
- **Editor background:** `#FFFFFF`
- **Sidebar background:** `#F3F3F3`
- **Activity bar:** `#2C2C2C` (stays dark even in light mode)
- **Foreground text:** `#000000` (editor), lighter in sidebars
- **Key insight:** Pure white editor, slightly gray sidebar, some elements (like activity bar) remain dark for contrast anchoring

### G. Figma
- **Canvas background (light mode default):** `#F5F5F5`
- **Panel backgrounds:** `#FFFFFF`
- **Key insight:** Very neutral, no strong tinting, relies on elevation/shadows for hierarchy

### H. Tailwind CSS Neutral Scale (reference)
- **50:** `#F9FAFB`
- **100:** `#F3F4F6`
- **200:** `#E5E7EB`
- **300:** `#D1D5DB`
- **400:** `#9CA3AF`
- **500:** `#6B7280`
- **600:** `#4B5563`
- **700:** `#374151`
- **800:** `#1F2937`
- **900:** `#111827`
- **Key insight:** Cool-neutral with very subtle blue-gray undertone. Most apps use this or Slate for light mode surfaces.

---

## 3. Universal Design Principles for Premium Light Mode

### Text Hierarchy Rules
| Role | Contrast Ratio | Example Hex | Usage |
|------|---------------|-------------|-------|
| Primary text | 12:1+ vs bg | `#1A1B22` to `#14151A` | Headings, main content |
| Secondary text | 7:1+ vs bg | `#3F4048` to `#4B5563` | Labels, setting titles |
| Muted text | 4.5:1+ vs bg | `#636674` to `#6B7280` | Descriptions, hints |
| Disabled text | 3:1 (min) | `#9A9CA5` to `#9CA3AF` | Disabled items |

### Background Layering Rules
| Layer | Purpose | Recommended Value |
|-------|---------|-------------------|
| App background | Deepest canvas | `#F5F6FA` or `#F4F4F7` (subtle blue-gray tint) |
| Sidebar | Navigation panel | Same or 1 step lighter than app bg |
| Card / Panel | Elevated content | `#FFFFFF` (pure white) |
| Input / Recessed | Text fields, wells | `#F0F1F5` to `#ECEEF2` |
| Hover state | Interactive feedback | `rgba(0,0,0,0.04)` overlay or `#F0F1F5` |
| Active/Selected | Currently chosen | `#EDE9FE` (purple-tinted) or `#E8E5FF` |

### Border Rules
| Type | Value | Usage |
|------|-------|-------|
| Card border | `#E5E7EB` or `rgba(0,0,0,0.08)` | Around cards |
| Separator | `#ECEEF2` or `rgba(0,0,0,0.06)` | Between items |
| Focused | Accent color at 40% | Input focus rings |
| Subtle | `#F0F1F5` | Between sidebar items |

### Accent Color Adaptation
- Purple accent stays the SAME in both themes (it's a brand color)
- But accent *backgrounds* (badges, pills) become much lighter/more pastel:
  - Dark mode accent bg: `#2D1B69` (deep)
  - Light mode accent bg: `#EDE9FE` or `#F3F0FF` (very pale lavender)
- Success green in light mode: `#DCFCE7` bg, `#15803D` text
- Error red in light mode: `#FEE2E2` bg, `#B91C1C` text

---

## 4. Step-by-Step Implementation Roadmap

### Step 1: Update Color & Brush Tokens (`Themes/LightTheme.xaml`)
*   Refine `TokenAppBg` to a premium cool-tinted off-white (`#F5F6FA`)
*   Refine `TokenCardBg` → `#FFFFFF` with soft border `#E2E4EA`
*   Refine `TokenInputDarkBg` → `#ECEEF2` (recessed look)
*   Ensure `TokenSurfaceBg` → `#EEEEF4` (segmented control bg)
*   Text tokens already good (`#14151A` primary, `#26272E` secondary)
*   Add proper accent backgrounds (pale lavender for purple category)

### Step 2: Main Dashboard Sidebar (`MainWindow.xaml`)
*   Apply the refined `TokenSidebarItemTextBrush` and active states.
*   Selected item: pale purple bg `#EDE9FE` + bold purple text `#6D28D9`
*   Hover: `rgba(0,0,0,0.04)` or `#F5F3FF`

### Step 3: Settings Panel Layout & Badges (`Views/SettingsDashboardView.xaml`)
*   Clean up card boundaries with proper light-mode border tokens
*   Ensure toggle controls, tooltips, and badges have refined light-mode borders
*   Zoom buttons: use `TokenTextSecondary` not hardcoded white

### Step 4: Validation
*   Check all WCAG AA contrast ratios (4.5:1 for body text, 3:1 for large text/icons)
*   Verify no invisible text on any panel
*   Test sidebar, cards, inputs, toggles, badges all look distinct and layered

---

## 5. Sources
- [Notion color system](https://github.com/soulcore-dev/soul-design-md/blob/main/designs/notion/DESIGN.md)
- [Discord light theme vars](https://gist.github.com/Epicpkmn11/573b7a1c4f5a039f39516f75e5c7b7fe)
- [Apple HIG system colors](https://gist.github.com/lithammer/e9e68c131297c3158a654c0fdfc4111a)
- [Radix Colors slate scale](https://gist.github.com/adamdotjs/98d6f46b75062c565946904ee618c465)
- [Linear design system](https://github.com/nexu-io/open-design/blob/main/design-systems/linear-app/DESIGN.md)
- [Tailwind neutral colors](https://tailwindcss.com/docs/colors)
- [Microsoft Fluent 2 tokens](https://fluent2.microsoft.design/color-tokens2/)
- Content was rephrased for compliance with licensing restrictions.


---

## 6. Final Implementation Status (2026-07-16)

### Theme System: Complete
5 themes implemented with clickable color circles in Settings:

| # | Name | Swatch | Type | AppBg |
|---|------|--------|------|-------|
| 1 | Midnight | `#1A1B23` | Dark (default) | `#101115` |
| 2 | Snowfall | `#ECEEF5` | Light (cool blue-gray tint) | `#ECEEF5` |
| 3 | Amethyst Dusk | `#2E2838` | Mid-tone twilight purple | `#2E2838` |
| 4 | Ocean Depth | `#0F1A2E` | Dark navy-blue | `#0A1628` |
| 5 | Soft Lavender | `#EDE6F7` | Light (warm purple tint) | `#EDE6F7` |

### Contrast Verified
- All themes pass WCAG AA for primary/secondary text
- Cards pop clearly against app backgrounds in every theme
- Borders visible on all surfaces
- No pure white backgrounds — both light themes are tinted
- Old Light/Dark toggle removed, replaced with theme circles

### Files Modified
- `ThemeService.cs` — 5-theme enum + GetThemeUri()
- `LightTheme.xaml` — refined contrast, cool tint
- `SoftLavenderTheme.xaml` — new, warm purple light
- `AmethystDuskTheme.xaml` — new, mid-tone twilight
- `OceanDepthTheme.xaml` — new, deep navy
- `SettingsDashboardView.xaml` — circle selector UI
- `SettingsDashboardViewModel.cs/.Commands.cs` — SelectThemeCommand
- `SettingsStyles.xaml` — DynamicResource fixes, GhostButton rewrite
- `App.xaml` — ComboBox dynamic colors


---

## 7. Session Decisions & Architecture Notes (2026-07-16)

### Theme System — Final Decision
- **2 themes only**: Midnight (dark, default) + Warm Stone (light)
- Old 5-theme system scrapped — users of a macro app don't care about 5 themes
- Light mode uses `#F0EDE8` warm gray/stone tone (NOT pure white)
- Toggle replaced with **2 clickable color circles** in Settings > User Interface > Appearance

### WebView2 Experiment — Paused
- **Goal**: Gradually replace WPF UI with HTML/CSS/JS via WebView2 for premium look
- **Architecture**: Keep C# backend (engine, hotkeys, AHK), only replace the visual layer
- **Resource strategy**: When app is open → web UI loads (some resources OK). When minimized/tray → kill UI, near-zero resources
- **What was built**: Dummy skeleton Macro Editor V2 (HTML with toolbar, step palette, timeline, properties panel)
- **Current status**: REMOVED from main app, saved to `_Experiments/WebUI_V2/`
- **Next step**: After stable version is committed/packaged, create a branch or copy for V2 experiments
- **Approach**: Gradual adoption — one view at a time, not a full rewrite

### Files in `_Experiments/WebUI_V2/`
- `MacroEditorV2/index.html` — full skeleton macro editor (toolbar, sidebar, timeline, properties)
- `Settings/index.html` — General settings panel (cards with toggles)
- Both use the app's dark theme colors as CSS variables

### Stable App — Current State
- All V2/experiment code removed from the main project
- App is clean and ready for commit/package
- Theme system works (2 themes, circle selector)
- Light mode contrast fixes applied across all major views
- GhostButton, ComboBox, toggle, checkbox styles all use DynamicResource

### Key Decision: WPF Settings Panel Stays
- Settings panel looks good in WPF — no need to rebuild in WebView2
- V2 experiment will focus on the **Macro Editor** view instead (where premium UI matters most)
