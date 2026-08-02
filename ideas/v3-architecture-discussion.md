# PowerX Keys V3 — Complete Architecture & Design Context

---

## 🏛️ V2 Baseline vs V3 Vision

### Current V2 Architecture
* **Frontend**: C# WPF + WinForms Hybrid (.NET 10.0).
* **Backend**: C# Services + SQLite DB (`macros.db`).
* **Engine**: AutoHotkey v2 Engine (`PowerX_Engine.exe`).
* **Dev Pain Point**: Any small UI tweak (color, margin, button position) requires stopping the app, rebuilding via `dotnet build`, and launching the executable. This creates slow iteration cycles.

### V3 Core Vision
* **Clean Scratch Rebuild**: Start fresh without carrying over legacy WPF architectural friction.
* **Hyper-Premium UI**: Focus on top-tier visual aesthetics (dark mode glassmorphism, fluid micro-animations, modern typography).
* **Developer Experience**: Instant UI Hot Reloading (HMR) so UI edits reflect immediately on screen without manual recompilation.
* **Balanced Resource Philosophy**: 
  * Modern PCs have 16GB+ RAM — using ~50–100 MB RAM while the UI window is actively open is a completely acceptable trade-off for a state-of-the-art interface.
  * When minimized to the system tray, resource consumption must drop to a **negligible / minimal level**.

---

## 🏗️ Architectural Options Evaluated

### 1. C# .NET 10 + WinUI 3 / WPF UI
* **Pros**: Native Windows 11 Fluent UI components, low background RAM (~15–30 MB).
* **Cons**: Lacks instant web hot-reload; requires standard C# rebuild cycles for UI layout changes.

### 2. Rust / C# + Tauri 2.0 (Web Frontend)
* **Pros**: Uses Windows native WebView2 runtime, 10MB installer size, full HTML/CSS/JS flexibility, instant HMR.
* **Tray RAM**: Drops to ~5–15 MB RAM when webview process is suspended.
* **Cons**: Requires Rust/C# bridge management.

### 3. Electron + Renderer Process Termination Strategy
* **Pros**: Easiest and fastest UI development, massive ecosystem, full design freedom, instant UI Hot Reloading.
* **Tray Mechanism**:
  * **Active Mode**: `Main Process` (~35 MB) + `BrowserWindow Renderer` (~100–150 MB).
  * **Tray Mode**: On window close/minimize to tray, destroy/close the `BrowserWindow` renderer process completely.
  * **Result**: Tray RAM drops from ~180 MB down to **~35–45 MB**.
  * **Restoration**: Double-clicking the tray icon re-instantiates the `BrowserWindow` (~0.5s load time).

### 4. C# .NET + WebView2 Host
* **Pros**: Preserves existing C# compiler/backend logic, loads HTML/CSS UI inside WebView2, supports `CoreWebView2.TrySuspendAsync()`.
* **Tray RAM**: ~5–15 MB RAM when suspended.

---

## 📊 Dual Logging Architecture (Dedicated Resource Auditor)

To guarantee empirical measurement of app performance during development and testing:

1. **Main App Log (`app.log`)**:
   * Records app lifecycle events, macro triggers, engine execution, DB transactions, and error stack traces.

2. **Dedicated Resource Audit Log (`resource_audit.log`)**:
   * **Scope**: Operates completely separately from the main log.
   * **Active Period**: Runs periodically *only* while the app process is active; automatically stops when the app closes.
   * **Metrics Recorded**:
     * App RAM Footprint (MB)
     * App CPU Utilization (%)
     * Active Thread Count
   * **Value**: Gives clear, un-biased empirical log data to verify exactly how much system resources the app used across different states (Active UI vs System Tray).

---

## 🔍 Research & Optimization Directive

* **Online Research Task**: When V3 implementation officially starts, conduct targeted online research to discover cutting-edge desktop/web workarounds (e.g. process suspension hacks, canvas memory offloading, background thread idling tricks) to push system tray usage even lower.
* **Empirical Verification**: Use `resource_audit.log` data during initial prototypes to evaluate candidate architectures before committing to a final production stack.

---

## 🤝 Multi-AI Collaboration & Launch Workflow

When the user gives the official **green signal** to start V3:

1. **Trigger `/factory` Workflow**: Run the Multi-AI Factory collaboration workflow to generate structured prompt files in `Obsidian Vault/prompts/`.
2. **Gather Agent Ideas**: Solicit UI design suggestions, component ideas, and layout options from all participating agents (Antigravity subagents, Kiro, etc.).
3. **Filter & Select**: Evaluate all agent proposals, filter out over-engineered noise, and select the best-of-the-best UI design blueprint.
4. **Interactive Dummy Prototype**: Build a standalone clickable UI prototype in `PowerX_Keys_V3/` (loaded in WebView2) with dummy data for user testing and visual approval.
5. **Incremental Implementation**: Once the prototype UI design is approved by the user, begin wiring up C# backend services step-by-step.

