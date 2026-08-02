---
tags: [feature, ui, easter-egg]
date: 2026-06-02
sources:
  - EasterEggTester\EasterEggWindow.xaml
  - EasterEggTester\EasterEggWindow.xaml.cs
  - PowerX_Keys_V2\Views\EasterEggWindow.xaml
  - PowerX_Keys_V2\Views\EasterEggWindow.xaml.cs
  - PowerX_Keys_V2\Services\EasterEggService.cs
  - PowerX_Keys_V2\MainWindow.xaml.cs
  - PowerX_Keys_V2\ViewModels\AIAssistantViewModel.cs
  - PowerX_Keys_V2\Managers\MacroDatabase.cs
  - PowerX_Keys_V2\Models\AppConfig.cs
status: implemented
---

# 🥚 Easter Eggs

**Summary:** 6 hidden secrets scattered throughout PowerX Keys. All use a unified golden RPG Achievement popup with animated trophy, progress bar, confetti, and sound.

## Architecture

### EasterEggService (`Services\EasterEggService.cs`)
- Centralized tracking — which eggs are found, stored in `SettingsModel.UnlockedEasterEggs`
- `TryUnlock(type)` → returns false if already found (prevents repeat popups)
- `TryUnlockAndShow(type, owner)` → unlock + show popup in one call
- `UnlockedCount` / `ProgressBarWidth` → used by popup for real progress

### EasterEggType Enum (`Models\AppConfig.cs`)
`VersionBadge`, `OldSchoolGamer`, `CuriousMind`, `NameDropper`, `WhisperInTheDark`, `ShiftClicker`

### Popup (`Views\EasterEggWindow.xaml` + `.cs`)
- Golden RPG Achievement card — unified for all 6 types
- Golden star ★ in glowing ring, bounce animation
- Title punch-in with scale effect
- Italic flavor text
- Progress bar fills to `UnlockedCount / 6`
- Counter: "X / 6 Secrets Found"
- Discovery stat: "Found by less than 1% of users"
- Golden confetti particles + tada.wav sound
- Unique button text per type (AWESOME!, GG!, etc.)

---

## ✅ All 6 Implemented

| # | Name | Trigger | Achievement Title | File |
|---|------|---------|------------------|------|
| 1 | Version Badge | Click `v5.3.0` badge 7 times | The Architect | `MainWindow.xaml.cs` |
| 2 | Konami Code | ↑↑↓↓←→←→BA anywhere | Old School Gamer | `MainWindow.xaml.cs` |
| 3 | AI Chat | Type "who made this" in AI chat | The Curious Mind | `AIAssistantViewModel.cs` |
| 4 | Macro Name | Name a macro "Maaz" | Name Dropper | `MacroDatabase.cs` |
| 5 | Type PowerX | Type "powerx" anywhere in app | Whisper In The Dark | `MainWindow.xaml.cs` |
| 6 | Logo + Shift | Shift+Click the app logo | The Shift Clicker | `MainWindow.xaml.cs` |

### Trigger Details

#### 1. Version Badge 7x Click → "The Architect"
- **Location:** `MainWindow.xaml.cs` → `VersionBadge_MouseDown()`
- **Flavor:** "Built from scratch, by one person. Every pixel, every line."

#### 2. Konami Code → "Old School Gamer"
- **Location:** `MainWindow.xaml.cs` → `OnPreviewKeyDown()`, 10-key buffer
- **Flavor:** "↑↑↓↓←→←→... Some codes never die."

#### 3. AI Chat → "The Curious Mind"
- **Location:** `AIAssistantViewModel.cs` → `SendMessageAsync()`, keyword detection
- **Keywords:** "who made this", "who created this", "who built this", "who is the developer"
- **Flavor:** "You asked the right question to the right person."
- Message still sends to AI normally — popup is a bonus

#### 4. Macro Name → "Name Dropper"
- **Location:** `MacroDatabase.cs` → `SaveMacro()`, after transaction commit
- **Check:** `macro.Name.Equals("Maaz", OrdinalIgnoreCase)`
- **Flavor:** "You spelled the creator's name. Coincidence? We think not."

#### 5. Type "powerx" → "Whisper In The Dark"
- **Location:** `MainWindow.xaml.cs` → `OnPreviewKeyDown()`, string buffer with 3s timeout
- **Flavor:** "You typed into the void. The void answered."

#### 6. Logo + Shift Click → "The Shift Clicker"
- **Location:** `MainWindow.xaml` → `PreviewMouseLeftButtonDown="AppLogo_MouseDown"`
- **Check:** `Keyboard.Modifiers == ModifierKeys.Shift`
- **Flavor:** "Hold Shift, click the logo. Simple, yet only the curious try."

---

## Testing Workflow

All popups are designed & tested in the **EasterEggTester** app first → once approved → copied to main app.

- Tester app: `EasterEggTester\`
- Main app: `PowerX_Keys_V2\`

---

## Key Files

- [EasterEggService.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Services/EasterEggService.cs) — Tracking service
- [EasterEggWindow.xaml](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/EasterEggWindow.xaml) — Popup layout
- [EasterEggWindow.xaml.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/EasterEggWindow.xaml.cs) — Popup logic
- [AppConfig.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Models/AppConfig.cs) — EasterEggType enum + UnlockedEasterEggs
- [MainWindow.xaml.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/MainWindow.xaml.cs) — Triggers 1, 2, 5, 6
- [AIAssistantViewModel.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/ViewModels/AIAssistantViewModel.cs) — Trigger 3
- [MacroDatabase.cs](file:///C:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Managers/MacroDatabase.cs) — Trigger 4

## Related Pages

- [[macro-editor]]
- [[script-library]]
- [[settings-dashboard]]
- [[ai-chat]]
