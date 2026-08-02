# PowerX Keys V3 — WebView2 Rewrite Plan

## Overview
After V2 is shipped and stable, start a fresh rebuild from scratch using WebView2 for the entire UI layer. Same app, same features, but modern architecture from day one.

## Core Idea
- New folder, clean start — no legacy code carried over
- C#/.NET backend handles: AHK compilation, process management, hotkeys, system tray, file I/O
- WebView2 frontend handles: All UI (HTML/CSS/JS or framework like React/Svelte)
- **Everything stays within the app window** — no external popups, no separate windows
- Proper modular architecture from the start (the 4-project split from `ideas/app-modularization.md`)

## Why
- Easier to build beautiful, responsive UI with web tech
- Theming/styling becomes trivial (CSS)
- Animations and transitions are smoother
- Easier for AI agents to work with (HTML is simpler than XAML)
- Cross-platform potential later (Electron/Tauri style)
- Clean codebase without years of WPF patches

## Architecture
```
PowerX Keys V3
├── PowerX.Core/          ← Models, DB, shared types
├── PowerX.Services/      ← AHK compiler, execution, hotkeys
├── PowerX.App/           ← C# host, WebView2 setup, system tray
└── PowerX.UI/            ← Web frontend (HTML/CSS/JS)
    ├── src/
    │   ├── components/   ← UI components
    │   ├── pages/        ← Macro editor, settings, dashboard
    │   └── bridge.js     ← C# ↔ JS communication layer
    └── dist/             ← Built output loaded by WebView2
```

## Key Rules
- No external windows — everything renders inside the main WebView2 panel
- C# ↔ JS communication via WebView2's `PostWebMessageAsJson` / `WebMessageReceived`
- Hotkey registration stays in C# (WinAPI level)
- AHK engine stays exactly the same (PowerX_Engine.exe)
- All overlays (capture, pixel pick) stay native C# (they need to be system-wide)

## Phases
1. **Shell** — Empty window with WebView2 loaded, basic navigation working
2. **UI Design** — Build the interface in web tech (match or improve V2's look)
3. **Bridge** — Wire up C# backend to JS frontend (macro CRUD, settings, hotkeys)
4. **Features** — Port all V2 features one by one
5. **Polish** — Animations, transitions, performance tuning
6. **Replace** — V3 becomes the main app, V2 enters maintenance mode

## Prerequisites Before Starting
- [ ] V2 fully packaged and released
- [ ] V2 stress tested and stable
- [ ] All V2 raw data/source preserved as backup
- [ ] This plan reviewed and approved

## UI Framework Candidates

| Option | Pros | Cons |
|--------|------|------|
| **WebView2** (recommended) | Native Microsoft, ships with Win 11, best .NET integration, suspend/resume API | Chromium instance (~60-80MB when active) |
| **Photino** | Super lightweight (~5MB), uses OS native webview, open source | Smaller community, less mature |
| **MAUI Blazor Hybrid** | Write UI in C# (Blazor), no JS needed, Microsoft supported | Heavier, MAUI has rough edges, complex setup |

**Winner: WebView2** — best C# interop, built-in suspend API, largest community, Win 11 has runtime pre-installed.

## Memory Management Strategy

The heavy UI memory is ONLY used when the user actively has the app open. Background/tray mode stays lean.

```
App visible (editing macros)  → WebView2 active (~60-80MB for UI)
App minimized or in tray      → CoreWebView2.TrySuspendAsync() → drops to ~5-10MB
App restored by user          → Resume → instant reload (local files)
```

Backend stays constant (~15-20MB): hotkey listener, AHK engine, system tray — same as V2.

**Key API:** `CoreWebView2.TrySuspendAsync()` / auto-resume on navigate. Discards render processes, tabs, and cached DOM while suspended. No user-visible delay on restore since we load from local disk.

## Notes
- V2 source stays intact — this is a parallel project, not a destructive rewrite
- Can reference V2's logic for the backend (copy compiler, execution service, etc.)
- UI is built fresh — no XAML conversion, design from scratch
- AI agents will assist throughout — document everything so any session can pick up
