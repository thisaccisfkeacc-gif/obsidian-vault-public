# PowerX Keys — Easter Egg System Health Check & Fix (Session Prompt)

Copy EVERYTHING below from the `--- COPY FROM HERE ---` marker to the end, and paste it into a new opencode session. The agent should follow it as a single task.

---

## --- COPY FROM HERE ---

You are working on a Windows desktop macro app called **PowerX Keys** (C# .NET, WPF + WinForms hybrid, AutoHotkey v2, SQLite, MVVM). Your job in this session is a **full health check of the "Easter Egg" system**, to fix every bug and inconsistency you find, and to leave the feature polished and consistent. Do a careful, thorough job — this is a quality pass, not a quick patch.

## FIRST — read these rules & context files (mandatory, in order)

1. `C:\Users\Maaz\Documents\New folder\AGENTS.md` — project rules & how to communicate (follow them strictly).
2. `C:\Users\Maaz\Documents\New folder\Obsidian Vault\KNOWLEDGE.md` — entry map, read before starting.
3. `C:\Users\Maaz\Documents\New folder\Obsidian Vault\core\GOTCHAS.md` — read before writing code.
4. `C:\Users\Maaz\Documents\New folder\Obsidian Vault\ideas\easter-egg-redesign.md` — design direction reference (ceremony, rarity, pips, vault). Use it as inspiration, but the primary requirement is that the system is healthy, consistent, and on-brand.
5. `C:\Users\Maaz\Documents\New folder\Obsidian Vault\wiki\log.md` — the log you must append to when done.

## THE BIG WARNING (read this twice)

**Another agent/user/editor has been modifying these files while work happens.** Files have been observed reverting or changing content between reads (e.g. a glyph path and a trigger were swapped). Therefore:

- **ALWAYS re-read the exact current on-disk content immediately before editing** (`EasterEggWindow.xaml`, `EasterEggWindow.xaml.cs`, `EasterEggService.cs`, `AppConfig.cs`, `MainWindow.xaml.cs`, `MainViewModel.cs`).
- Do NOT trust the descriptions below as the "final truth of disk" — treat them as a map and re-verify.
- Grep the repo for `EasterEgg` and `EasterEggType` first to get the complete, current list of every touch point.

## Build & run (use these exact commands)

- Kill the app first if running: `taskkill /f /im "PowerX Keys.exe"`
- Build: `dotnet build "C:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\PowerX_Keys_V2\PowerX_Keys_V2.csproj" -nologo -clp:ErrorsOnly`
- Expected: 0 errors. (Warnings exist pre-existing ~200+, ignore them unless your change adds new ones.)
- Run: `dotnet run --project "C:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\PowerX_Keys_V2\PowerX_Keys_V2.csproj"`
- Rebuild once after fixing; if `PowerX Keys.exe` is running/locking the build, kill it without asking (rule already allowed).

## What the Easter Egg system is

A secret-hidden achievement system. When a user discovers a secret, a special "Achievement Unlocked" popup window plays a short ceremony (flash → card reveal → trophy → confetti → sound) showing the secret's name, flavor text, and global progress out of 6.

### Current files (paths under `C:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\`)

| File | Role |
|---|---|
| `PowerX.UI\Views\EasterEggWindow.xaml` | The popup UI (XAML) |
| `PowerX.UI\Views\EasterEggWindow.xaml.cs` | Popup logic: per-egg texto, glyph art, rarity/theme, confetti, sound, copy, animations |
| `PowerX.UI\EasterEggService.cs` | Persistence + unlock + progress math (TotalEasterEggs = 6) |
| `PowerX.Core\Models\AppConfig.cs` | `EasterEggType` enum (~line 212) + `UnlockedEasterEggs: List<string>` (~line 209) |
| `PowerX_Keys_V2\MainWindow.xaml.cs` | Triggers: VersionBadge (7 clicks), Konami code, type "powerx", Shift+click logo |
| `PowerX.UI\ViewModels\AIAssistantViewModel.cs` | Trigger: "who made this" → CuriousMind |
| `PowerX.UI\ViewModels\MainViewModel.cs` | Trigger: star-filter 5 clicks → "NameDropper" (NEW — semantics changed recently, see findings) |
| `PowerX.Services\Services\ServicesUIHooks.cs` + `PowerX.UI\ServicesUIHooksUI.cs` | Hook wiring `UnlockEasterEggHook` → `EasterEggService.TryUnlockAndShow` |

### Current 6 secrets (re-verify each trigger in code)

1. **VersionBadge** — click the version badge 7×
2. **OldSchoolGamer** — Konami code ↑↑↓↓←→←→BA
3. **CuriousMind** — AI chat: ask "who made/made this" (heuristic in AIAssistantViewModel)
4. **NameDropper** — NOTE: comment says "Name macro 'Maaz'" but the ACTUAL trigger in MainViewModel.cs seems to have been changed to "5 fast clicks on the star filter". VERIFY this mismatch and pick ONE canonical meaning; make glyph, rarity label, flavor text, week — everything consistent with it. Fix `AppConfig.cs` enum comment to match reality.
5. **WhisperInTheDark** — type "powerx" anywhere on the dashboard
6. **ShiftClicker** — Shift + click the app logo

## KNOWN FINDINGS to review & fix (verify each; fix what's actually broken)

### Data / state
- **Concurrent-modification protection**: if a file's content changed since the described state above, re-base your edits on the actual content. If you find TWO conflicting versions of content in two files (e.g. a trigger in two places), consolidate to ONE and remove the dead one.
- `UnlockedEasterEggs` is a plain `List<string>`. Consider (only if quick & safe) adding a `UnlockedAtUtc` timestamp record + migration so a future "Secrets Vault" can show find-dates. If it risks data/JSON breakage, leave it and add a note in the wiki log instead.
- `EasterEggService.TryUnlock` is not thread/session safe. Two triggers could fire near-simultaneously (e.g. Konami + typing powerx at the same time). Guard with a lock and re-check-after-unlock, or at least document.
- The `UnlockEasterEggHook` invokes `TryUnlockAndShow` — verify it can be called from a non-UI thread (it should dispatch). Confirm the popup opens with the correct owner.

### EasterEggWindow.xaml / .xaml.cs (re-verify current on-disk state — fields may change)
- **Glyph consistency**: each egg should have its own vector glyph (`GlyphData` dictionary, `Path` element, `Geometry.Parse`). The "NameDropper" entry was observed holding the STAR path instead of its own — if it's still wrong, give it the **"M"/monogram glyph** (or whatever fits the trigger meaning) so no two eggs share the same art.
- **Rarity + theme**: app theme = dark navy + **purple primary** (#A78BFA / TokenPurple300) with **gold as the premium-only accent**. The popup should read as PURPLE-first, gold reserved for the "mythic" secret only. Re-map any remaining gold/high-contrast colors so each egg's accent uses the app token palette (see tokens below).
  - Token palette: Amber500 `#F59E0B`, Gold `#FBBF24`, Purple300 `#A78BFA`, Purple400 `#A855F7`, Purple500 `#8B5CF6`, Indigo300 `#818CF8`, Blue400 `#5AC8FA`, Green500 `#22C55E`, Rose300 `#FB7185`, Border `#2E1F4E`/`#4A3418`-dark edge tones.
  - Suggested mapping (align with current or pick better if inconsistent): Architect=Amber/Gold (mythic), Gamer=Purple, Curious=Indigo, NameDropper=Rose, Whisper=Blue, ShiftClicker=Green.
- **Rarity stop**. Verify the pill shows only the rarity name (not a hardcoded count like "1/6" — that was a bug; the real count should come from `EasterEggService.UnlockedCount` in the counter line only). If the pill ever hardcodes a count again, remove it.
- **Progress pips (6)**: reassigned `UpdateProgressPips` fills the pips up to UnlockedCount; the newly unlocked pip should be visually distinct (pulse/glow), others neutral. If missing, add it.
- **Copy Card button**: it copies text then showed a MessageBox → it should now flash an inline label ("Copied ✓") WITHOUT a MessageBox. Verify; if a MessageBox is back, remove it.
- **Drag behavior**: the window is draggable via any left-click; buttons (✕, Copy, primary) must NOT start a drag (check the `Window_MouseDown` filter). Verify clicking buttons doesn't drag.
- **Escape**: Esc closes. Keep.
- **Close (✕)**: top-right ghost close button. Keep.
- **Sound**: `PlaySound()` was moved to the UI thread because SoundPlayer on a background thread plays nothing. VERIFY the current code does this correctly (no `Task.Run` wrapping) and the fallback chain is `tada.wav` → `notify.wav` → `SystemSounds.Asterisk`. Also ensure `_soundPlayer` is disposed on close and not double-played.
- **Confetti**: particles should be removed on animation completed; max tick cap; no leak if the window closes mid-party. Verify.
- **Focus/a11y**: on open the primary button should be focusable; Esc works; text sizes ≥ 11px; contrast reasonable (secondary text ~#9CA3AF level on the card).

### Animations & performance
- All motions should use `RenderTransform` + `Opacity` only (no animating `Width`/`Height`/`Margin`).
- Brushes are pre-frozen in a static ctor (keep that pattern — don't allocate brushes per tick).
- If the window is killed mid-animation, timers must be stopped in `OnClosed` (it does stop/conjecture — verify).

### Verification you MUST do
- After fixing: run the app, **reset the eggs to re-test**: edit `%LOCALAPPDATA%/PowerXKeys/AppConfig.json`, remove the `UnlockedEasterEggs` array, save, relaunch, then trigger each secret and confirm: window opens (topmost, centered on owner), animation plays, sound plays (tada), Esc closes, ✕ closes, copy shows "Copied ✓" then resets, drag works on empty area but not on buttons, pips fill correctly.
- Trigger the hardest ones via code review if you can't practically: Konami via the keyboard on the dashboard, and "powerx" by typing in the dashboard (they work). AI heuristic and star-filter you can test manually.
- Build must be 0 errors after your changes.

## App theme reference (so you stay on-brand)
- Raycast Premium dark palette: base `~#14121` `#14121E`, surfaces `#14121E`/`#0F1017`, purple primary purple300/400/500, gold premium accents. Any color you add should come from the token list above or a light/dark variant.
- Do NOT change the app-wide styles or tokens unless necessary — the wrap in this one window.

## Deliverables
1. All findings fixed and build verified (0 errors).
2. Append a concise summary to `C:\Users\Maaz\Documents\New folder\Obsidian Vault\wiki\log.md` (throttled rule: you may skip if the last log was <15 min ago, but note it at the end).
3. When you finish, answer with a **short, friendly, non-technical summary** (per AGENTS.md): what you checked, what you fixed, anything you deliberately left or couldn't test, and any new ideas for the future (the "Secrets Vault").

## Rules
- **Short, simple, no jargon when talking to the user.** Technical detail goes in the wiki log.
- Never edit `.ahk` scripts. Don't touch `_Archive/`, `Obsidian Vault/raw/`, `bin/obj/`.
- If something is uncertain (a deliberate-looking choice, a conflict), flag it — do not silently over-engineer.
- Don't spawn subagents/skills unless truly needed; the audit is within a handful of files.
- When done, stop — don't keep "improving" beyond the task without permission.

---

## --- END OF PROMPT ---