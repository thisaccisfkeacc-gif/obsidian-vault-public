# Light Theme Conversion — Job Board

READ THIS FIRST, ENTIRE FILE, BEFORE TOUCHING ANYTHING.

## What this is
PowerX Keys (WPF app) just got a Light Theme system added. Two files already exist and define every color you'll need:
- `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Themes/DarkTheme.xaml`
- `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Themes/LightTheme.xaml`

Both define the SAME resource keys (just different hex values). Your job is to find hardcoded hex colors in specific view files and replace them with `{DynamicResource TokenXxxBrush}` references to the closest matching key in those two files.

**READ BOTH THEME FILES IN FULL BEFORE STARTING ANY JOB.** Do not guess key names.

## How this board works (IMPORTANT — read carefully)
- Below is a list of JOBS. Each job has a status: `OPEN`, `IN PROGRESS (agent: <name>)`, or `DONE (agent: <name>)`.
- When you (an agent) pick a job:
  1. Pick the FIRST job still marked `OPEN`.
  2. Immediately edit this file and change that job's status to `IN PROGRESS (agent: <pick a short name for yourself, e.g. Agent-1, Agent-2, etc — must be unique, check no one else used it below>)`. Save the file. This claims the job so no one else does it.
  3. Do the work described in that job (see RULES below).
  4. When done, edit this file again: change the job status to `DONE (agent: <your name>)` and fill in the "Result" line under that job with a one-line summary (files touched, # of colors converted, any hex values you couldn't map).
  5. Go back to step 1 and pick the next OPEN job. Repeat until no jobs are OPEN.
- If you ever see a job already `IN PROGRESS` or `DONE`, skip it — do not redo it.
- If ALL jobs are `DONE`, write a line at the very bottom under "## All Done" saying so, and stop.

## RULES for every job (same for all agents, all files)
- Every hardcoded hex color (`Background="#xxxxxx"`, `Foreground="#xxxxxx"`, `BorderBrush="#xxxxxx"`, `Fill="#xxxxxx"`, `Stroke="#xxxxxx"`, `GradientStop Color="#xxxxxx"`, etc.) → replace with the closest matching token.
  - For Brush-type properties (Background, Foreground, BorderBrush, Fill, Stroke, etc.): use `{DynamicResource TokenXxxBrush}`.
  - For `GradientStop Color="..."` (needs a Color, not a Brush): use `{DynamicResource TokenXxx}` (the Color key, no "Brush" suffix), e.g. `Color="{DynamicResource TokenPurple300}"`.
- ALWAYS use `DynamicResource`, NEVER `StaticResource`, for these — so the theme can swap live without restart.
- Brand accent colors (purple #A78BFA/#A855F7/#8B5CF6/#9333EA family, blue #288DE5/#5AC8FA, green #4ADE80/#22C55E/#10B981, red #FF4D4D/#F43F5E/#EF4444, orange #FDBA74/#FF9F0A, indigo #818CF8, rose #FB7185, cyan #00E5FF) → map to the matching TokenPurple300Brush / TokenBlue500Brush / TokenGreen300Brush / etc. Match by CLOSEST hex value, don't guess semantically.
- Pure white `#FFFFFF` used as normal body/UI text → `TokenTextPrimaryBrush`. Pure white used as decorative icon/text ON TOP of a colored/accent background (e.g. white text on a purple SAVE button) → leave as `White` or `#FFFFFF`, don't convert.
- Do NOT touch: FontFamily, FontSize, Opacity values (unless it's literally a color's alpha baked into hex like #33xxxxxx — those are fine to convert if a close token match exists, otherwise leave), Data path strings (icon geometry), StrokeDashArray, layout/size numbers.
- Do NOT invent new token keys that don't exist in the theme files.
- If you can't confidently map a specific hex value, LEAVE IT AS-IS and note it in your Result line. Don't guess.
- `ColorAnimation`/`DoubleAnimationUsingKeyFrames` targeting `(SolidColorBrush.Color)` or `(LinearGradientBrush.GradientStops)[n].Offset` inside `<Storyboard>` — these can be left as hardcoded hex. Animating a DynamicResource-bound brush's Color this way often breaks WPF's freezing. Leave animation `To=`/`Value=` hex colors untouched unless you're confident it's safe.
- Preserve structure/formatting. Only change color values.
- After editing a file, RE-READ it to confirm the edit applied and the XAML is still well-formed (balanced tags, no typos in resource keys).
- Do NOT run `dotnet build`. Someone else will build everything at the end.
- Do NOT touch any `.xaml.cs` code-behind files. XAML only.

---

## JOBS

### Job 1 — MacroEditorView.xaml
File: `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/MacroEditorView.xaml`
Status: DONE (agent: Antigravity)
Result: Converted 168 colors to DynamicResources, mapped opacity-based background/border colors to nested SolidColorBrushes.

### Job 2 — MacroStepCard.xaml
File: `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/MacroStepCard.xaml`
Status: DONE (agent: Antigravity)
Result: Converted 122 colors to DynamicResources, mapped opacity-based background/border logic blocks, left transparent white placeholders untouched.

### Job 3 — MacroStepTemplates.xaml
File: `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/MacroStepTemplates.xaml`
Status: DONE (agent: Antigravity)
Result: Converted 989 colors in the 5 split templates merged by MacroStepTemplates.xaml, mapped opacity-based orange/green colors and yellow gradients, left WPF FallbackValue/TargetNullValue hardcoded hexes untouched.

### Job 4 — MacroEditorOverlays.xaml + MacroEditorCheatSheet.xaml
Files:
- `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/MacroEditorOverlays.xaml`
- `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/MacroEditorCheatSheet.xaml`
Status: DONE (agent: Antigravity)
Result: Converted 163 colors to DynamicResources across both files, leaving pure white setters untouched.

### Job 5 — CustomActionCard.xaml + RecordingWidgetView.xaml
Files:
- `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/CustomActionCard.xaml`
- `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/RecordingWidgetView.xaml`
Status: DONE (agent: Antigravity)
Result: Converted 350 colors across both files to DynamicResources, mapped warning pill opacity background/borders, left white foreground setters untouched.

### Job 6 — ScriptLibraryView.xaml
File: `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/ScriptLibraryView.xaml`
Status: DONE (agent: Antigravity)
Result: Converted 270 colors to DynamicResources, left white foreground/transparent indicators untouched.

### Job 7 — SettingsDashboardView.xaml (PART 1 — first half of the file)
File: `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/SettingsDashboardView.xaml`
Note: This file is VERY large. For this job, only convert the FIRST HALF of the file (roughly line 1 to the file's total-line-count/2 — check total lines first, e.g. with a line-count read, then work top to bottom until you're about halfway). Stop partway through a top-level section boundary (don't cut in the middle of a Border/Style block) rather than exactly at the midpoint line number.
Status: DONE (agent: Antigravity)
Result: Converted 255 colors in the first half of the file, leaving white swatches and indicators untouched.

### Job 8 — SettingsDashboardView.xaml (PART 2 — second half of the file)
File: `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/SettingsDashboardView.xaml`
Note: This file is VERY large. Job 7 (a different agent) is converting the FIRST half. You convert the SECOND half (from roughly the midpoint to the end). Before starting, read the file and check whether Job 7 is already `IN PROGRESS` or `DONE` — if Job 7 hasn't started yet, still proceed with your half but be careful: re-read the file right before you save any edit, in case Job 7's agent is editing concurrently. Make your edits in small chunks (one Style/Border section at a time) and re-read after each save to avoid clobbering Job 7's work.
Status: DONE (agent: Antigravity)
Result: Converted 170 colors in the second half of the file, leaving white foreground setters untouched.

### Job 9 — SettingsView.xaml + TextSnippetsView.xaml
Files:
- `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/SettingsView.xaml`
- `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/TextSnippetsView.xaml`
Status: DONE (agent: Antigravity)
Result: Converted 75 colors in SettingsView.xaml and 141 colors in TextSnippetsView.xaml to DynamicResources, left white foreground elements untouched.

### Job 10 — AIAssistantView.xaml
File: `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/AIAssistantView.xaml`
Status: DONE (agent: Antigravity)
Result: Converted 76 colors to DynamicResources, leaving white text setters untouched.

### Job 11 — Dialog batch A (message/crash/warning dialogs)
Files:
- `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/DarkMessageBoxWindow.xaml`
- `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/CrashReportWindow.xaml`
- `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/SafetyWarningDialog.xaml`
- `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/HardwareLockWarningDialog.xaml`
Note: Some of these already have PARTIAL conversion done (mix of `StaticResource` and hardcoded hex). Check each file carefully — any `StaticResource TokenXxx` you find should be changed to `DynamicResource TokenXxx` (same key, just swap the binding type). Then convert remaining hardcoded hex on top of that.
Status: DONE (agent: Antigravity)
Result: Swapped 6 StaticResources -> DynamicResources, converted 60 colors to DynamicResources across the four dialog xaml files.

### Job 12 — Dialog batch B (input/naming/picker dialogs)
Files:
- `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/NamingConflictDialog.xaml`
- `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/InputPromptWindow.xaml`
- `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/DropdownPromptWindow.xaml`
- `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/ExportMacroPickerDialog.xaml`
- `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/ProfileCreationWindow.xaml`
Status: DONE (agent: Antigravity)
Result: Swapped 10 StaticResources -> DynamicResources, converted 128 colors to DynamicResources across five dialog xaml files.

### Job 13 — Dialog batch C (image/capture tool windows)
Files:
- `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/CoordinateEditDialog.xaml`
- `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/ImagePreviewWindow.xaml`
- `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/ImageStudioWindow.xaml`
- `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/CaptureLibraryWindow.xaml`
Status: DONE (agent: Antigravity)
Result: Converted 129 colors to DynamicResources across four dialog xaml files, leaving literal red preview and transparent overlay elements untouched.

### Job 14 — ForceUpdateWindow.xaml + final sweep
Files:
- `PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/ForceUpdateWindow.xaml`
Then: search the `Views/` folder (excluding `.xaml.cs` files, and excluding these purpose-built dark overlay/tool files which should stay untouched: `CaptureOverlay.xaml`, `MagnifierPreview.xaml`, `SplashWindow.xaml`, `EasterEggWindow.xaml`, `KeyCaptureWindow.xaml`, `CoordinatePickerWindow.xaml`, `WindowPickerWindow.xaml`, `TargetPickerWindow.xaml`, `OffsetCaptureWindow.xaml`) for any remaining `.xaml` file not covered by Jobs 1-14 above and not in that exclusion list. If you find one with hardcoded hex colors, convert it using the same rules. List any such extra files you handled in your Result line.
Status: DONE (agent: Antigravity)
Result: Converted 9 colors to DynamicResources in ForceUpdateWindow.xaml, verified final sweep across Views/ folder (no other unmapped XAML files).

---

## All Done
All jobs complete — confirmed by Antigravity on 2026-07-16T13:51:00Z.
