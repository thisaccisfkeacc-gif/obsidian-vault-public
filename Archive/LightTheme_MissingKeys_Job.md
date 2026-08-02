# Job — Find & Fix Missing Theme Resource Keys

## Status: OPEN

## Background
PowerX Keys got a Light Theme system added. All colors live in two files that must mirror each other key-for-key:
- `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Themes/DarkTheme.xaml`
- `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Themes/LightTheme.xaml`

During the color conversion pass, some views ended up referencing `{DynamicResource TokenXxxBrush}` keys that were **never actually defined** in either theme file. WPF fails silently when a resource key doesn't exist — the element just renders invisible/black/wrong. This is why some icons, containers, and toggle indicators look broken in the Settings screens (bug icon, tip jar/coffee container, mobile remote container, "auto reload" green check indicator, etc).

Already fixed as of this note: added a batch of missing brushes to both theme files (TokenPurple100Brush, TokenPurple200Brush, TokenPurple600Brush, TokenPurple700Brush, TokenPurpleBorderBrush, TokenBlue300Brush, TokenBlue400Brush, TokenBlueBgSubtleBrush, TokenBlueBorder2Brush, TokenGreen400Brush, TokenGreen500Brush, TokenGreen600Brush, TokenGreenBgSubtleBrush, TokenGreenBorderBrush, TokenGreenBgLightBrush, TokenRed400Brush, TokenRed500Brush, TokenRedBgSubtleBrush, TokenRedBorderStrongBrush, TokenRedPressedBrush, TokenOrangeBgBrush, TokenOrangeBorderBrush, TokenIndigoBgBrush, TokenIndigoBorderBrush, TokenRoseBgBrush, TokenRoseBorderBrush, TokenLogicYesBrush, TokenLogicNoBrush, TokenTextGhostBrush).

## The Task
1. Read both theme files IN FULL. List every `x:Key` defined in `DarkTheme.xaml` (Color, SolidColorBrush, LinearGradientBrush). Confirm `LightTheme.xaml` has the exact same set of keys — note any mismatch.
2. Search every `.xaml` file under `Views/`, plus `MainWindow.xaml` and `App.xaml`, for all occurrences of `DynamicResource TokenXxx` and `StaticResource TokenXxx`.
3. Cross-reference: any Token* key referenced in a view but NOT defined in `DarkTheme.xaml` is a **missing key** — this is the bug causing invisible/broken elements.
4. For every missing key:
   - Figure out if it needs to be a `Color` or a `SolidColorBrush` based on how it's used (Fill/Background/Foreground/BorderBrush = Brush; `GradientStop Color=` = Color).
   - Add it to BOTH `DarkTheme.xaml` and `LightTheme.xaml`, near related keys, following the existing pattern exactly (e.g. `<SolidColorBrush x:Key="TokenXxxBrush" Color="{StaticResource TokenXxx}"/>`).
   - Reuse an existing base Color if one already exists with the same name minus "Brush". Only pick a new hex value if genuinely nothing close exists — and if so, keep brand accent colors (purple/blue/green/red/orange/indigo/rose/cyan) IDENTICAL between Dark and Light theme, only surface colors (backgrounds/borders/text) should differ between the two files.
5. Re-read both theme files after editing to confirm they're well-formed XML and have matching key sets.
6. Do NOT modify any file under `Views/`, `MainWindow.xaml`, or `App.xaml` — only add missing definitions to the two theme files.
7. Do NOT run `dotnet build`.

## When done
Edit this file: change `Status: OPEN` to `Status: DONE (agent: <your name>)`, and add a `## Result` section below listing every missing key you found, added, and which files referenced them.

## Result
(fill in when done)
