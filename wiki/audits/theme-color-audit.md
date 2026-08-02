# Theme Color Audit: Hardcoded Dark Mode Colors

**Audit Date:** July 25, 2026
**Scope:** PowerX.UI + PowerX_Keys_V2 XAML files
**Type:** Read-only scan — no code changes

---

## 📌 Status
- ✅ **23 New Color Tokens Created**: All 23 missing tokens (Amber, Close hover/pressed, Blue hover, Red light text, Indigo selection) have been added to both `DarkTheme.xaml` and `LightTheme.xaml`.

---

## Summary

| Area | Files Scanned | Hardcoded Colors |
|------|---------------|------------------|
| App.xaml | 1 | 100 |
| MainWindow.xaml | 1 | 56 |
| Views (PowerX.UI) | 28 | 292 |
| **TOTAL** | **30** | **448** |

**Worst offenders:** App.xaml (100), MainWindow.xaml (56), SettingsDashboardView.xaml (48), AuthWindow.xaml (43), CaptureOverlay.xaml (44)

---

## Most Repeated Hardcoded Colors

| Hex | Count | Should Be |
|-----|-------|-----------|
| `#FFFFFF` | 50+ | `TokenTextPrimaryBrush` |
| `#A78BFA` | 25+ | `TokenPurple300Brush` |
| `#4ADE80` | 8 | `TokenGreen300Brush` |
| `#F59E0B` | 10 | Need `TokenAmber500` |
| `#1A1C24` | 8 | `TokenBorderSubtleBrush` |
| `#0F1013` | 10 | `TokenCardBgBrush` |
| `#C8CAD4` | 6 | `TokenTextTertiaryBrush` |
| `#8C8D94` | 8 | `TokenTextDimBrush` |

---

## Missing Tokens (Need Creation)

| Token Name | Hex | Used For |
|------------|-----|----------|
| `TokenAmber300` | `#FCD34D` | Orange button border |
| `TokenAmber500` | `#F59E0B` | Orange/Premium buttons |
| `TokenAmber600` | `#D97706` | Orange gradient end |
| `TokenCloseHoverBg` | `#A62626` | Close button hover |
| `TokenClosePressedBg` | `#F1707A` | Close button pressed |
| `TokenBlueHover` | `#359DF5` | Toggle hover state |
| `TokenBlueBorderHover` | `#2583D0` | Toggle border hover |
| `TokenRedTextLight` | `#FFE4E9` | Active pill text |
| `TokenIndigoSelection` | `#3730A3` | TextBox selection |

---

## WPF Limitation

**15 values** are inside `ColorAnimation To="..."` — WPF Storyboard `ColorAnimation` only accepts literal Color values, NOT `{DynamicResource}`. These must remain hardcoded OR the animation needs restructuring.

---

## StaticResource That Should Be DynamicResource

> **Update 2026-08-01:** All four rows below are **RESOLVED** — verified no longer using `StaticResource`:

| File | Line (old) | Current | Status |
|------|-----------|---------|--------|
| App.xaml | 319 | `{DynamicResource GreenRunBackgroundBrush}` at App.xaml:304 | ✅ Already DynamicResource |
| App.xaml | 320 | `{DynamicResource GreenRunForegroundBrush}` at App.xaml:305 | ✅ Already DynamicResource |
| App.xaml | 709 | No `{StaticResource BorderSubtleBrush}` remains in App.xaml | ✅ Resolved |
| MacroEditorView.xaml | 423 | `{DynamicResource PanelBackgroundBrush}` at MacroEditorView.xaml:408 | ✅ Already DynamicResource |

## Verified Line References (2026-08-01)

- **GhostButton** style is defined at **App.xaml:687** (and again at `SettingsStyles.xaml:164` — the duplicate-definition issue stands, with updated refs).
- Hardcoded colors remain at **App.xaml:408** (`#F59E0B` PremiumOrangeButton gradient start), **App.xaml:420** (`#F59E0B` inside `ColorAnimation`), **App.xaml:933** (`#A78BFA` ComboBox ItemTemplate foreground).

---

## Full Findings by File

### App.xaml (100 hardcoded values)

| Line | Hex | Style | Attribute |
|------|-----|-------|-----------|
| 281 | `#1E1A29` | RunButton | Background |
| 282 | `#A78BFA` | RunButton | Foreground |
| 296 | `#2D2342` | RunButton | Animation |
| 342 | `#303138` | ManageButton | Background |
| 343 | `#E0E2EE` | ManageButton | Foreground |
| 380 | `#8B5CF6` | BlueManageButton | Background |
| 381 | `#FFFFFF` | BlueManageButton | Foreground |
| 417 | `#9333EA` | PremiumPurpleButton | Background |
| 418 | `#FFFFFF` | PremiumPurpleButton | Foreground |
| 425 | `#A855F7` | PremiumPurpleButton | BorderBrush |
| 405 | `#FCD34D` | PremiumOrangeButton | BorderBrush |
| 408 | `#F59E0B` | PremiumOrangeButton | Gradient |
| 409 | `#D97706` | PremiumOrangeButton | Gradient |
| 497 | `#1E1A29` | InactivePillButton | Background |
| 498 | `#E9D5FF` | InactivePillButton | Foreground |
| 499 | `#4C2B80` | InactivePillButton | BorderBrush |
| 550 | `#2B1A1E` | ActivePillButton | Background |
| 551 | `#FFE4E9` | ActivePillButton | Foreground |
| 552 | `#802B3A` | ActivePillButton | BorderBrush |
| 603 | `#2A171A` | RecordPillButton | Background |
| 604 | `#FF4D4D` | RecordPillButton | Foreground |
| 605 | `#4C2224` | RecordPillButton | BorderBrush |
| 643 | `#288DE5` | BlueActivePillButton | Background |
| 644 | `#FFFFFF` | BlueActivePillButton | Foreground |
| 669 | `#656773` | SidebarHeaderStyle | Foreground |
| 703 | `#1A1C21` | CreateMacroButtonGradient | Background |
| 704 | `#FFFFFF` | CreateMacroButtonGradient | Foreground |
| 714 | `#25272D` | CreateMacroButtonGradient | Trigger |
| 726 | `#1A1C21` | PremiumActionButtonStyle | Background |
| 727 | `#2D2F36` | PremiumActionButtonStyle | BorderBrush |
| 808 | `#8C8D94` | WindowCloseButton | Foreground |
| 818 | `#A62626` | WindowCloseButton | Trigger |
| 819 | `#FFFFFF` | WindowCloseButton | Trigger |
| 822 | `#F1707A` | WindowCloseButton | Trigger |
| 994 | `#288DE5` | CompactToggleSwitchStyle | Trigger |
| 995 | `#1A73C0` | CompactToggleSwitchStyle | Trigger |
| 997 | `#FFFFFF` | CompactToggleSwitchStyle | Trigger |
| 1024 | `#A78BFA` | PremiumComboBoxStyle | Trigger |
| 1061 | `#656773` | PremiumComboBoxStyle | Stroke |
| 1102 | `#13161F` | ParamTextBoxStyle | Background |
| 1103 | `#F8FAFC` | ParamTextBoxStyle | Foreground |
| 1104 | `#2B3245` | ParamTextBoxStyle | BorderBrush |
| 1108 | `#818CF8` | ParamTextBoxStyle | CaretBrush |
| 1139 | `#15161A` | DarkParamComboBoxStyle | Background |
| 1140 | `#25262B` | DarkParamComboBoxStyle | BorderBrush |
| 1141 | `#E1E3E8` | DarkParamComboBoxStyle | Foreground |
| 1146 | `#1C1D21` | ComboBoxItem | Background |
| 1156 | `#25272D` | ComboBoxItem | Trigger |
| 1159 | `#2A2F3D` | ComboBoxItem | Trigger |
| 1160 | `#A78BFA` | ComboBoxItem | Trigger |
| 1170 | `#1A1D24` | PremiumAddButtonStyle | Background |
| 1171 | `#A0A2AE` | PremiumAddButtonStyle | Foreground |
| 1207 | `#656773` | ResetButton | Foreground |
| 1225 | `#FF4D4D` | ResetButton | Trigger |
| 1226 | `#25272D` | ResetButton | Trigger |
| 1327 | `#D1D3D9` | PremiumCheckboxStyle | Foreground |
| 1354 | `#1E1A2E` | PremiumCheckboxStyle | Trigger |

### MainWindow.xaml (56 hardcoded values)

| Line | Hex | Attribute |
|------|-----|-----------|
| 51 | `#A78BFA` | BorderBrush |
| 136 | `#A78BFA` | Stroke |
| 175 | `#8B8E99` | Fill |
| 200 | `#8B8E99` | Foreground |
| 214 | `#EF4444` | Fill |
| 314 | `#A855F7` | Background |
| 375 | `#10B981` | Fill |
| 447 | `#A855F7` | Background |
| 486 | `#10B981` | Fill |
| 521 | `#A78BFA` | Foreground |
| 526 | `#EF4444` | Foreground |
| 568 | `#A855F7` | Background |
| 573 | `#10B981` | Fill |
| 603 | `#FFFFFF` | Foreground |
| 609 | `#E1E3E8` | Foreground |
| 693 | `#A0A2AE` | Foreground |
| 709 | `#EF4444` | Foreground |
| 777 | `#F59E0B` | Foreground+Border |
| 812 | `#BF5AF2` | Foreground |
| 835 | `#5A5C67` | Foreground |
| 873-877 | various | Sparkle colors |
| 897 | `#FACC15` | Fill |
| 912 | `#38BDF8` | Foreground |
| 929 | `#D946EF` | Fill |
| 944 | `#A78BFA` | Foreground |
| 975 | `#D9070A10` | Background |
| 1000 | `#16141F` | Color |
| 1023 | `#26233A` | Stroke |
| 1025 | `#A78BFA` | Stroke |
| 1032 | `#FFFFFF` | Foreground |
| 1036 | `#7F828E` | Foreground |

### AuthWindow.xaml (43 hardcoded values)

| Line | Hex | Attribute |
|------|-----|-----------|
| 21 | `#0C0D10` | Background |
| 22 | `#E8EAF2` | Foreground |
| 23 | `#A78BFA` | CaretBrush |
| 24 | `#4C2B80` | SelectionBrush |
| 25 | `#21222C` | BorderBrush |
| 47 | `#35364A` | Trigger |
| 50 | `#7C3AED` | Trigger |
| 51 | `#0E0F14` | Trigger |
| 76 | `#7C3AED` | Gradient |
| 77 | `#9D5CF6` | Gradient |
| 102-104 | various | Background/FG/Border |
| 127-130 | various | Google brand (keep) |
| 138-142 | various | Triggers |
| 154-156 | various | Background/FG/Border |
| 177 | `#8B8FA8` | Foreground |
| 183-187 | various | Triggers |
| 200 | `#484A5C` | Foreground |
| 218 | `#C0392B` | Trigger |
| 222 | `#962D23` | Trigger |
| 233 | `#6B6E84` | Foreground |
| 249 | `#A78BFA` | Foreground |
| 263 | `#0F1013` | Background |
| 283 | `#F0F1FA` | Foreground |
| 289 | `#5A5D72` | Foreground |
| 295-296 | `#1F0C0C`/`#5C1B1B` | Error bg |
| 306 | `#FF6B6B` | Foreground |
| 326 | `#8B5CF6` | Foreground |
| 369-370 | various | Fill/Background |
| 399 | `#6B6E84` | Foreground |
| 424 | `#D0D2E0` | Foreground |

### SettingsDashboardView.xaml (48 hardcoded values)

| Line | Hex | Attribute |
|------|-----|-----------|
| 57 | `#FFFFFF` | Trigger |
| 160 | `#FFFFFF` | Fill |
| 790 | `#FFFFFF` | Background |
| 930 | `#1A1B23` | Fill |
| 950 | `#F0EDE8`/`#CCC8C0` | Fill/Stroke |
| 1144-1162 | `#FFFFFF` | Triggers (4x) |
| 1809 | `#B07070` | Foreground |
| 1827-1828 | `#FFFFFF` | Triggers |
| 2653-2654 | `#2D1060`/`#6D28D9` | Gradient |
| 2658 | `#D8B4FE` | Foreground |
| 2705 | `#122D20` | Background |
| 2708 | `#231550` | Background |
| 2717 | `#4ADE80` | Foreground |
| 2721 | `#A78BFA` | Foreground |
| 2752-2780 | various | Amber section |
| 2812 | `#A78BFA` | Foreground |
| 2829 | `#0C0D12` | Background |
| 2854 | `#22232E` | Background |
| 2865-2869 | various | Triggers |
| 2916-2933 | various | Logout section |

### WindowPickerWindow.xaml (35 hardcoded values)

| Line | Hex | Attribute |
|------|-----|-----------|
| 15 | `#E2E8F0` | Foreground |
| 28-33 | various | Triggers |
| 43-46 | various | Background/FG/Border |
| 64-74 | various | Triggers |
| 96-99 | various | Triggers |
| 109 | `#0D0E15` | Background |
| 124-125 | various | Foreground |
| 131-150 | various | Text colors |
| 171 | `#161720` | Background |
| 181 | `#F1F5F9` | Foreground |
| 193 | `#0A0B0E` | Background |
| 205-211 | various | Text colors |
| 217 | `#8B5CF6`/`#FFFFFF` | Button |
| 231-238 | various | Triggers |

### CaptureOverlay.xaml (44 hardcoded values)

Most are structural overlay colors (black/white with alpha) — ~15 are tokenizable.

| Line | Hex | Attribute |
|------|-----|-----------|
| 9 | `#288DE5` | Stroke |
| 51-65 | `#4ADE80`/`#6C6E76` | Green accent/text |
| 89 | `#FF4ADE80` | Stroke |
| 97-108 | various | Green section |
| 119-152 | various | Blue/Red sections |

### EasterEggWindow.xaml (22 hardcoded values)

Special event window — uses amber/gold palette not in theme system.

| Line | Hex | Attribute |
|------|-----|-----------|
| 51-60 | various | Background gradients |
| 89-91 | `#F59E0B`/`#F97316` | Gradients |
| 94-105 | `#F59E0B` | Foreground |
| 115-117 | `#FFFFFF`/`#F59E0B` | Gradient |
| 129-130 | various | Background |
| 135 | `#FDBA74` | Foreground |
| 145 | `#08090C` | Background |
| 153-154 | `#F59E0B`/`#F97316` | Gradient |

### Other Views (remaining hardcoded values)

| File | Count | Notes |
|------|-------|-------|
| SplashWindow.xaml | 9 | Purple themed |
| SkeletonOverlay.xaml | 11 | Shimmer effect |
| MagnifierPreview.xaml | 17 | Crosshair colors (structural) |
| OffsetCaptureWindow.xaml | 10 | Purple accent |
| MacroEditorView.xaml | 9 | Animation colors |
| KeyCaptureWindow.xaml | 5 | Purple/Red accent |
| MacroStepCard.xaml | 5 | White on accent |
| CustomActionCard.xaml | 3 | White on accent |
| ProfileCreationWindow.xaml | 3 | White on accent |
| CrashReportWindow.xaml | 2 | White on accent |
| DarkMessageBoxWindow.xaml | 2 | White on accent |
| InputPromptWindow.xaml | 2 | White on accent |
| AIAssistantView.xaml | 2 | White on accent |
| ImageStudioWindow.xaml | 3 | White + color picker |
| CoordinateEditDialog.xaml | 1 | White on accent |
| DropdownPromptWindow.xaml | 1 | White on accent |
| MacroEditorCheatSheet.xaml | 1 | White on accent |
| MacroEditorOverlays.xaml | 1 | White on accent |
| MouseTemplates.xaml | 2 | White on accent |
| CaptureLibraryWindow.xaml | 1 | Overlay |
| CoordinatePickerWindow.xaml | 1 | Overlay |
| SettingsView.xaml | 3 | White icons |

---

## Labels Key

- ✅ **Tokenizable** — Can be replaced with theme token
- ⚠️ **Animation** — Inside ColorAnimation, cannot use DynamicResource
- 📌 **Intentional** — Structural color (black/white/alpha) or brand color
- ❌ **Missing Token** — Needs new token created first

---

**Last Updated:** August 1, 2026
