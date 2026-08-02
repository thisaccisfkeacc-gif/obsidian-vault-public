# Subscription Expired Window Edge Case Scan

Files reviewed: `SubscriptionExpiredWindow.xaml`, `SubscriptionExpiredWindow.xaml.cs`, `App.xaml.cs` (show/launch site), `SupabaseAuthService.cs` (`IsSubscriptionValid`)

## Findings

### Medium
1. **Log out fails → app stays open but window freezes** – If `SignOutAsync()` throws (network error), the catch fires, `MessageBox.Show` displays, but `Process.Start` + `Shutdown()` are skipped (they're after `await` in the `try` block). The window stays visible, but user can't retry logout without closing and reopening.
2. **Log out restart loop** – Logout restarts app via `Process.Start` + `Shutdown()`. New instance has no session, subscription is invalid, shows the same window again. User must click "Exit" to break loop. Not a bug per se, but infinite loop if user keeps clicking "Log out".
3. **No `Owner` set** – `App.xaml.cs:494` creates window without `Owner`. `ShowDialog()` works but modal behavior isn't bound to any window. `Topmost=True` mitigates but if splash hadn't closed (it does), dialog could go behind.
4. **No drag handle / not movable** – `WindowStyle="None"` and `AllowsTransparency="True"` with no `MouseLeftButtonDown` handler. User cannot drag the window if it appears off-screen (e.g., on a secondary monitor that disconnects).
5. **No Enter/Escape keyboard handling** – Enter doesn't trigger "Upgrade to Premium", Escape doesn't close. Mouse-only interaction.

### Low
6. **Redundant `Topmost=true`** – Set in both XAML (line 10) and code-behind (`App.xaml.cs:495`). Harmless.
7. **`BuyButton_Click` silently swallows all exceptions** – `catch { }` with no logging. If URL fails to open (sandboxed env, no browser), user gets no feedback.
8. **Hardcoded URL** – `PaymentUrl = "https://powerxkeys.com/#pricing"` in code-behind. No staging/test variant.
9. **`CloseButton_Click` calls `this.Close()` not `Shutdown()`** – Works because `App.xaml.cs:499` calls `Shutdown()` after `ShowDialog()` returns. But if window were ever opened non-modally, "Exit" would close window but leave app running.
10. **`Process.Start(System.Environment.ProcessPath)`** – Spawns new process, but `Application.Current.Shutdown()` runs immediately after. New process may not have fully initialized before old one terminates. Usually works, but race-y.
