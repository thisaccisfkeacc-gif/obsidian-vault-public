# Auth Window Edge Case Scan

Files reviewed: `AuthWindow.xaml`, `AuthWindow.xaml.cs`, `AuthViewModel.cs`

## Findings

### Critical (Likely Bug)
1. **License key textbox hardcoded to `False` for spellcheck and text wrapping** – fine, intentional.
2. **`PasswordBox.Password` binding missing in XAML** – `LoginViewModel.Password` property exists, but the XAML `<PasswordBox>` never binds to it. Instead it's read via `x:Name="PasswordBox"` in code-behind, which works but breaks MVVM and prevents validation from updating. Not a crash but a design gap.
3. **No `MaxLength` on license key or password fields** – no overflow risk but could allow very long input.

### Medium
4. **`AuthWindow.xaml.cs` manually sets `this.DataContext` in constructor** – ViewModel is never set via DI or XAML `<Window.DataContext>`, so any future DI changes break auth.
5. **Window `ShowInTaskbar="False"` and `WindowStyle="None"` but `Topmost="True"`** – modal to main window but no `Owner` assignment. Could appear behind other windows if main window isn't focused.
6. **Enter key not handled** – user must click login button; no `KeyDown` handler for Enter.
7. **No loading spinner / disable state** – login button stays enabled during auth attempt; user could double-click and fire two requests.
8. **Error messages use `MessageBox.Show`** instead of inline validation; dismisses naturally but feels jarring.

### Low
9. **No remember-me / persist license** – user must re-enter every launch.
10. **Hardcoded window size `Width="420" Height="380"`** – not responsive.
11. **No `WindowStartupLocation="CenterScreen"`** – default is manual, should center.
12. **No `Closed` event cleanup** – no unsubscription from events if ViewModel uses them.
13. **Min-width/min-height not set** – could resize to zero if borderless grip permits.
14. **`AuthWindow` never set as `Owner` of main window** – if user clicks main window, auth goes behind.

### AutoHotkey Hook
15. **AuthWindow's `PasswordBox` receives input before AHK hook initializes** – no known race, but AHK's keyboard hook could theoretically capture password keystrokes before login window has focus.
