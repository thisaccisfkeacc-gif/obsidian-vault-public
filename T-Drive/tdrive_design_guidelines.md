# Professional UI/UX Design Guidelines — T-Drive

Reference document for achieving a premium, professional dark-mode file manager interface. Not generic AI slop — these are real patterns from Google Drive, Dropbox, Apple Finder, and top SaaS products.

---

## 1. Dark Mode Done Right

### Colors
- **Never use pure black** (`#000000`). Use dark gray with subtle tonal quality: `#1a1a2e`, `#0f172a`, `#1e1e2e`.
- **Text primary:** Off-white `#e2e8f0` or `#f1f5f9` — never `#ffffff` (causes eye strain).
- **Text secondary:** Muted gray `#94a3b8` or `#64748b`.
- **Accent:** One primary color, desaturated for dark backgrounds. `#06b6d4` (cyan) works. Avoid overly saturated colors.
- **Surfaces:** Use elevation through lightness, not shadows. Cards are slightly lighter than background. Modals are lighter than cards.

### Elevation System (Material Design 3)
| Element | Background | Example |
|---|---|---|
| Page background | Darkest | `#090d16` |
| Sidebar | Slightly lighter | `#0f172a` |
| Card / Surface | Lighter still | `#1e293b` |
| Card hover | Lightest surface | `#334155` |
| Modal overlay | Dark + blur | `rgba(0,0,0,0.6)` + `backdrop-filter: blur(8px)` |

### Shadows in Dark Mode
Shadows are nearly invisible on dark backgrounds. Use:
- **Subtle border** instead: `1px solid rgba(255,255,255,0.06)`
- **Border color change on hover** for interactivity feedback
- **Box shadow only on elevated elements** (modals, dropdowns): `0 8px 32px rgba(0,0,0,0.4)`

---

## 2. Typography

### Font Stack
```css
font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
```
Inter is the industry standard for SaaS dashboards. Clean, geometric, excellent at small sizes.

### Type Scale
| Element | Size | Weight | Color |
|---|---|---|---|
| Page title | 1.5rem (24px) | 700 | Text primary |
| Section header | 1.125rem (18px) | 600 | Text primary |
| File name | 0.875rem (14px) | 600 | Text primary |
| File metadata | 0.75rem (12px) | 400 | Text secondary |
| Breadcrumb | 1.25rem (20px) | 700 | Text primary (current) / secondary (parent) |
| Button text | 0.875rem (14px) | 500 | White on accent / Text on outline |

### Rules
- Line height for body text: 1.6-1.7 (slightly higher in dark mode for readability)
- Letter spacing: -0.01em to -0.02em for headings (tighter = more premium)
- Never use font-weight 300 or 400 for headings — looks thin and weak

---

## 3. Spacing & Layout

### Spacing Scale (4px base)
```
4px  — xs
8px  — sm
12px — md
16px — lg
20px — xl
24px — 2xl
32px — 3xl
48px — 4xl
```

### Card Grid
```css
.file-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    gap: 16px;
}
```
- Min card width: 200px (grid), 180px (tight)
- Gap: 16px (not 20px or 24px — too much whitespace looks sparse)
- Card padding: 16px (consistent with gap)

### Sidebar Width
- Fixed: 260px (current is good)
- Compact: 200px (icon-only mode for power users)
- Collapsible on mobile: transform translateX(-100%)

---

## 4. File Card Design — The Most Important Element

### What Professional Cards Look Like
```
┌─────────────────────────┐
│                         │  ← Thumbnail area (120-140px height)
│      [thumbnail]        │     Rounded corners on top only
│                         │     Background: rgba(0,0,0,0.2)
├─────────────────────────┤
│ file_name.pdf           │  ← Name: 14px, weight 600, 1-line clamp
│ 2.4 MB  ·  Jan 15      │  ← Meta: 12px, weight 400, secondary color
│                         │     Separator: middle dot · not dash
├─────────────────────────┤
│ 👁  ✏️  📤  ⬇️  🗑️     │  ← Actions: ONLY on hover (never always visible)
└─────────────────────────┘
```

### Critical Rules
1. **Actions hidden by default.** Show on hover only. Always-visible actions clutter the card and look amateur.
2. **Thumbnail takes 60-70% of card height.** The visual preview is the primary value.
3. **Name truncation:** `text-overflow: ellipsis; white-space: nowrap; overflow: hidden;` — max 1 line.
4. **Metadata uses middle dot separator:** `2.4 MB · Jan 15, 2026` — not `2.4 MB | Jan 15` or `2.4 MB, Jan 15`.
5. **Hover state:** Subtle border color change + slight translateY(-2px) + shadow. NOT background color change (looks cheap).
6. **Border radius:** 12px for cards (not 10px or 16px — 12px is the sweet spot).
7. **Folder cards:** Different visual treatment — icon-based, no thumbnail, subtle accent color on border or icon.

### Hover Animation
```css
.file-card {
    transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
    border: 1px solid rgba(255,255,255,0.06);
}
.file-card:hover {
    border-color: rgba(255,255,255,0.12);
    transform: translateY(-2px);
    box-shadow: 0 8px 24px rgba(0,0,0,0.3);
}
```

---

## 5. Topbar / Toolbar

### Google Drive Pattern
- Search bar: Centered, rounded (24px radius), max-width 720px, with search icon
- Actions: Right-aligned, minimal icons with tooltips
- Sort: Dropdown, not always visible — toggle via icon

### T-Drive Should Be
```
[🔍 Search files in T-Drive...]     [🟢 T-Drive] [🔄] [Sort ▾] [⊞ ═]
```
- Search: Centered, pill-shaped (border-radius: 24px), subtle background
- Status badge: Small, left of actions
- Actions: Icon-only buttons, 32-36px size
- View toggle: Segmented control (active state has filled background)

---

## 6. Sidebar

### What Looks Professional
- **Logo area:** Clean, not oversized. Icon + text, 24px icon.
- **Upload button:** Full-width, gradient, prominent. Keep this.
- **New Folder button:** Outline style, full-width, below upload. Keep this.
- **Nav items:** Left-aligned, icon + text, 14px, weight 500. Active state: left border accent + subtle background.
- **Footer:** Storage info + Lock button. Keep this.
- **Spacing:** 24px between sections (not more).

### What Looks Cheap
- ❌ Too many colors in sidebar
- ❌ Oversized icons
- ❌ Inconsistent padding
- ❌ No active state indicator
- ❌ Hover state that changes background completely

---

## 7. Modal / Preview Design

### Current Problem
The preview modal shows the file name but the content area is empty or shows the wrong thing. The modal is too small and has no visual hierarchy.

### Professional Modal
```
┌──────────────────────────────────────────────┐
│  ✕                                           │  ← Close button: top-right, 32px
│                                              │
│  image_592.png                               │  ← Title: 18px, weight 600
│  3 KB · Uploaded Jul 25, 2026                │  ← Meta: 14px, secondary
│                                              │
│  ┌──────────────────────────────────────────┐│
│  │                                          ││  ← Preview area: fills modal
│  │           [image preview]                ││     max-height: 70vh
│  │                                          ││     object-fit: contain
│  │                                          ││
│  └──────────────────────────────────────────┘│
│                                              │
│  [⬇ Download]  [✏ Rename]  [📤 Share]       │  ← Actions: bottom, horizontal
└──────────────────────────────────────────────┘
```

### Rules
- Modal max-width: 850px (images), 600px (other)
- Backdrop: `rgba(0,0,0,0.7)` + `backdrop-filter: blur(12px)`
- Border radius: 16px
- Padding: 24px
- Close button: Absolute top-right, not inside the content flow
- Image preview: `max-width: 100%; max-height: 70vh; object-fit: contain; border-radius: 8px;`

---

## 8. Empty States

### What Looks Professional
```
        [📁 icon, 64px, muted color]

     No files in your T-Drive

  Drag and drop files here or click Upload
  to get started with your cloud storage.

        [ Upload Your First File ]
```

### Rules
- Icon: 64px, muted/accent color, no animation (or subtle pulse)
- Heading: 20px, weight 600, text primary
- Description: 14px, weight 400, text secondary, max-width 400px, centered
- CTA button: Primary style, centered
- Padding: 80px top/bottom (generous breathing room)

---

## 9. Breadcrumbs

### Google Drive Pattern
```
My Drive  ›  Projects  ›  2026  ›  Report
```
- Current item: Bold, accent color
- Parent items: Normal weight, secondary color, clickable
- Separator: `›` (not `/` or `>` — looks cleaner)
- Font size: Match page title (20px, weight 700)

---

## 10. Upload Progress

### What Looks Professional
- Floating card at bottom-right (not top of content area)
- Shows: file name, progress bar (real, not fake), percentage, speed, ETA
- Stays visible during upload, auto-hides after 3 seconds on completion
- Multiple uploads: Stack vertically with max 3 visible, "+X more" indicator

### What Looks Cheap
- ❌ Full-width progress bar at top of content
- ❌ Fake pulsing animation (no real progress)
- ❌ No file name shown
- ❌ Disappears immediately on completion

---

## 11. Key Anti-Patterns to Avoid

| Cheap Look | Professional Look |
|---|---|
| Pure black background | Dark gray with blue/warm tint |
| Pure white text | Off-white (#e2e8f0) |
| Always-visible action buttons | Hover-only actions |
| Flat cards with no depth | Subtle border + hover elevation |
| Centered everything | Left-aligned content, centered modals |
| Oversized icons (48px+) | Consistent 24-32px icons |
| Rainbow accent colors | 1-2 accent colors max |
| No transitions | Smooth 200ms cubic-bezier transitions |
| Fixed layouts | Responsive grid with auto-fill |
| Loading spinners everywhere | Skeleton screens for content areas |
| Alert() for confirmations | Custom modal confirmations |
| Border-radius: 4px or 999px | Border-radius: 8-12px (sweet spot) |

---

## 12. Reference Designs to Study

| Project | What to Learn |
|---|---|
| Google Drive (dark mode) | Card grid, search bar, breadcrumb, hover states |
| Dropbox (web) | File previews, sharing UI, empty states |
| Apple Finder (macOS) | Column view, list view, selection patterns |
| Linear.app | Dark mode done perfectly, typography, spacing |
| Vercel Dashboard | Minimal UI, professional dark theme |
| Raycast | Dark mode, command palette, keyboard navigation |
| Arc Browser | Sidebar design, spatial feel |

---

*This document is a living reference. Update as design decisions are made.*
