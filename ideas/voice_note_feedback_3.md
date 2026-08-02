# 🎧 Voice Note Feedback & Task Breakdown — Part 3

**Source:** User UI/UX Feedback & Task Review Session  
**Date:** July 29, 2026  

---

## 📋 Task & Bug Backlog


### 3. 🎨 Auto-Detect, Gear Menus & Visual Polish
    - Fix missing Recapture/Search Area button rendering after capturing pixel color.
  - Match capture flow with editor (don't force search area dialog).


LOGS:
### Bugs Fixed (100% Certain)

1. **App crash when typing >1 char in Output Text field** (`TextSnippetsView.xaml:511`)
    
    - Changed `UpdateSourceTrigger=PropertyChanged` → `LostFocus` to prevent binding re-entrancy crash with `Emoji.Wpf.RichTextBox`
2. **Remove "Select from List" button** (4 instances across 3 files)
    
    - Removed from `CustomActionCard.xaml:1494`, `TextSnippetsView.xaml:706`, `ScriptLibraryView.xaml:1327` & `1690`
3. **Broken gear visibility in Auto-Detect modes** (`CustomActionCard.xaml:914,1080,1283`)
    
    - Fixed 3 broken bindings: `HasCapturedImage`/`Pixel`/`UIElement` → `HasTriggerImage`/`Pixel`/`UIElement` (the `HasCaptured*` properties only exist on `MacroItem`, not `ActionItem`)
4. **Emoji Preview Glitch** (`CustomActionCard.xaml:290`)
    
    - Added `Text="{Binding Icon}"` binding to `CustomEmojiTextBox` so the picker popup always shows the current icon
5. **Context Menu "Disable" hover glitch & weird icons** (`TextSnippetsView.xaml:388-421`)
    
    - Replaced emoji in `Header` text (✅ ⛔ 📋 ⬆ ⬇ 🗑) with proper WPF `MenuItem.Icon` elements using Segoe MDL2 Assets font
6. **Garbled Unicode characters across XAML** (6 instances)
    
    - Fixed `âš¡` → `&#x26A1;` (⚡) in Capture App buttons
    - Fixed `â›¶` → `&#xE895;` (refresh icon) in Recapture buttons
    - Fixed `âŒ•` → `&#xE768;` (search icon) in Limit Search Area buttons
7. **QR Code not scannable** (`SettingsDashboardViewModel.cs:1035`)
    
    - Changed `drawQuietZones=false` → `true` so Google Lens can identify QR code boundaries