# Logo Spin Animation Edge Case Scan

Files reviewed: `MainWindow.xaml.cs` (lines 559-677), `MainWindow.xaml` (lines 98-105)

**Behavior:** Normal click → 360° spin (0.55s) + 8 sparkle particles burst out and fade. Shift+click → Easter egg unlock.

## Findings

### Medium
1. **`_logoAnimating` permanently stuck if any exception occurs** – `SpawnLogoSparkles()` called after `_logoAnimating = true`. If `TranslatePoint` fails (logo not loaded, zero size), or any sparkle creation throws, the `catch` isn't there. `_logoAnimating` stays `true` forever — logo never spins again until app restart.
2. **No rate limit on rapid normal clicks** – `_logoAnimating` prevents re-triggers during animation. But rapid clicks between animations stack multiple Canvas overlays. Each click adds a new Canvas to `MainGrid`. Sparkles from the first click clear themselves, but no upper limit on concurrent canvases.
3. **Canvas overlay leaks if animation callback never fires** – If the fade `DoubleAnimation` is GC'd or never completes, `canvas` stays in `MainGrid.Children` permanently. No cleanup fallback.
4. **Shift+click has no rate limit** – Spamming Shift+click calls `EasterEggService.TryUnlockAndShow` on every click. Depending on that method's guard, it could spam UI or logic.

### Low
5. **`e.Handled = true` set before `_logoAnimating` guard** – Even blocked clicks are marked handled. Minor, only affects the logo element.
6. **Attached property animations** – `BeginAnimation(Canvas.LeftProperty, ...)` on attached properties has known WPF quirks. Could silently fail to animate on some .NET versions.
7. **`Canvas.LeftProperty` animation not relative** – Animate from fixed `logoPos.X` to `endX`. If the logo moves during animation (window resize, DPI change), particles don't track.
8. **`RenderOptions.BitmapScalingMode="HighQuality"` on logo** – Minor perf cost for 28×28 image. Noticed.
