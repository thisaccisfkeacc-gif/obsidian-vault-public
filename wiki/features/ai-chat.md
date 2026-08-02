---
tags: [feature, ai, openrouter, chat, macro-generation, ui]
date: 2026-06-04
sources:
  - Views/AIAssistantView.xaml
  - ViewModels/AIAssistantViewModel.cs
  - MainWindow.xaml
status: current
---

# AI Chat (PowerX Intelligence) 🤖

The AI Assistant is a **floating popup window** that lets users interact with AI to get help and **generate macros from natural language**. It connects using the built-in free multi-provider fallback chain.

## Overview

- **Floating Popup Window** — Opens as a separate overlay popup anchored to the header's `START` button, keeping the main window dimensions static and unaffected.
- **Tabs-Based Selector** — Prominent "Chat Assistant" (purple indicator) and "Build Macro" (green indicator) tabs at the top of the panel replace the previous input ComboBox.
- **Copy-to-Clipboard** — Hover-revealed copy button (`&#xE8C8;`) on assistant message bubbles.
- **UI Input Cleanup** — Refined text input box with the mode selector removed.
- **Multi-model fallback** for 100% uptime.
- **"Inject Macro" button** auto-saves generated macros into the app.
- **Daily quota tracking** with remaining generations displayed in the header.

## Architecture

### UI Layer (`AIAssistantView.xaml` & `MainWindow.xaml`)
- **Floating Popup**: A `<Popup>` element in `MainWindow.xaml` with `PlacementTarget` set to the `StatusBannerButton` (START button). Opens/closes via the `IsAIAssistantOpen` binding. The popup has `StaysOpen="True"` and `AllowsTransparency="True"`, sized at `480x700`.
- **Mode Selection Tabs**: Horizontal `RadioButton` controls styled as premium tabs located right below the header. The active tab displays an colored underline indicator matching the theme.
- **Hover Copy Button**: A copy-to-clipboard button defined inside `AssistantMessageTemplate` that is hidden by default and becomes visible when the cursor hovers over the message container (`MsgBorder.IsMouseOver` trigger).
- **Message Templates**: User messages (right, gradient background) + assistant messages (left, dark background).
- **Markdown Rendering**: Rendered via `MarkdownHelper`.
- **Loading & Status Indicators**: Pulsing dot animation when `IsBusy` is true.
- **BETA Badge** in the header.

### ViewModel (`AIAssistantViewModel`)
- Manages `Messages` collection (user + assistant `AIChatMessage` objects).
- **Mode properties**: `IsChatMode` and `IsBuildMode` wrap `SelectedModeIndex` for two-way tab bindings.
- **Copy Command**: `CopyCommand` copies message text to `System.Windows.Clipboard`.
- `SendCommand` sends user input to the OpenRouter/Supabase Edge Function proxy.
- `InjectMacroCommand` parses AI JSON output → creates `MacroItem` → saves to database.
- `RetryCommand` re-sends the last failed prompt.

## Multi-Model Fallback

The AI engine cycles through multiple models to guarantee uptime:
1. Llama 3.3
2. Qwen
3. Gemma
4. Nemotron

## Macro Generation (Build Mode)

When in Build mode:
1. User describes a macro in plain English.
2. AI returns structured JSON with macro steps.
3. The response automatically live-builds/injects steps if they are in the Macro Editor, or auto-saves and binds it to a hotkey otherwise.

## Key Files

- [AIAssistantView.xaml](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Views/AIAssistantView.xaml)
- [AIAssistantViewModel.cs](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/ViewModels/AIAssistantViewModel.cs)
- [MainWindow.xaml](file:///c:/Users/Maaz/Documents/New%20folder/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/MainWindow.xaml)

## Related Pages

- [[settings-dashboard]]
- [[macro-editor]]
- [[macro-item]]
