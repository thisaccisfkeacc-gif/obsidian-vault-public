---
tags: [audit, architecture, security, performance]
date: 2026-07-23
author: Antigravity Agent
status: completed
---

# 🛡️ PowerX Keys — Deep Codebase Audit & Security Findings

> **Auditor:** Antigravity AI Agent  
> **Scope:** Architecture, WPF Performance, Security & API Hardening, Async/Threading Safety, and Database I/O in `PowerX_Keys_V2`.

---

## Executive Summary

This audit complements existing findings with **deep-dive structural, security, and memory safety analysis** across the codebase. Key critical findings include **unauthenticated LAN file uploads**, **20 process-crashing `async void` command handlers**, **global session token collision**, and **SQLite disk I/O bottlenecks**.

---

## 1. 🔒 Security & API Hardening

### 1.1 Unauthenticated Soundboard File Upload (`POST /api/soundboard/upload`)
* **Location:** `Services/RemoteServerService.cs` — line 1341
* **Risk:** High (Resource Exhaustion / Unauthorized File Write)
* **Finding:** While other endpoints check `if (!IsAuthorized(context)) return;`, the `/api/soundboard/upload` route omits authorization entirely. Any device connected to the local Wi-Fi network can upload sound files (up to 5 MB per payload) directly to `%LOCALAPPDATA%/PowerXKeys/Soundboard/` without entering the remote PIN.
* **Remediation:** Add `if (!IsAuthorized(context)) return;` at the top of the route handler.

### 1.2 Single Session Token Collision
* **Location:** `Services/RemoteServerService.cs` — line 144
* **Risk:** Medium (Session Hijacking / Disruption)
* **Finding:** `_sessionToken` is stored as a single `string` field on the service instance. When a second mobile device or browser tab authenticates with the correct PIN, `_sessionToken` is overwritten with a new `Guid`. This instantly invalidates the session of the first device, causing connection drops in multi-device households.
* **Remediation:** Replace single `_sessionToken` with a thread-safe `ConcurrentDictionary<string, DateTime>` to support multi-device session tokens with expiration timestamps.

### 1.3 Missing CORS Headers & CSRF Protection
* **Location:** `Services/RemoteServerService.cs` — `SendJson` method
* **Risk:** Medium (Cross-Site Request Forgery)
* **Finding:** `SendJson` does not set explicit CORS headers (`Access-Control-Allow-Origin`), nor does the server handle HTTP `OPTIONS` preflight requests. Furthermore, browser-based remotes rely solely on query parameter tokens `?token=...`, making requests vulnerable to CSRF if a user visits a malicious website on the same network.
* **Reference:** [OWASP REST Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html)

---

## 2. ⚡ WPF Performance & Memory Safety

### 2.1 Process Crash Hazard: 20 `async void` Methods
* **Locations:** ViewModels & Event Handlers (e.g., `ExecuteCaptureAppBound`, `ExecuteRunMacro`, `ExecuteCaptureScreenEvent`)
* **Risk:** High (Application Stability)
* **Finding:** 20 command execution and helper methods are declared as `async void`. In .NET, exceptions thrown in `async void` methods cannot be caught by caller `try/catch` blocks or outer task wrappers. Any unhandled exception will bubble straight to the `SynchronizationContext` and crash the entire WPF application process (`AppDomain.UnhandledException`).
* **Remediation:** Convert all `async void` methods (except top-level XAML event handlers) to `async Task` or wrap internal code in comprehensive `try/catch` blocks.
* **Reference:** [Async/Await - Best Practices in Asynchronous Programming (Microsoft Learn)](https://learn.microsoft.com/en-us/archive/msdn-magazine/2013/march/async-await-best-practices-in-asynchronous-programming)

### 2.2 Event Subscription Memory Leaks
* **Location:** ViewModels subscribing to static events (e.g., `ConfigManager.Current`, `TrayIconManager.Instance`)
* **Risk:** Medium (Memory Growth)
* **Finding:** ViewModels and overlay windows subscribe to singleton events using `+=` without implementing `IDisposable` or unsubscribing in `Unloaded` handlers. This holds strong references to short-lived ViewModels, preventing Garbage Collection (GC).
* **Remediation:** Implement `WeakEventManager` or explicitly unsubscribe (`-=`) in window/view close handlers.

### 2.3 GDI & Drawing Resource Management
* **Location:** Screenshot capture & Overlay window rendering (`RemoteServerService.cs`, `CaptureOverlay.xaml.cs`)
* **Risk:** Medium (GDI Handle Exhaustion)
* **Finding:** High-frequency screen captures create `Bitmap` and `Graphics` objects rapidly. If GDI objects are not disposed promptly, Windows can reach the 10,000 GDI handle limit, leading to UI rendering glitches or `OutOfMemoryException`.

---

## 3. 🗄️ Database & I/O Optimizations

### 3.1 SQLite Transaction & WAL Mode Overhead
* **Location:** `Managers/MacroDatabase.cs`
* **Risk:** Medium (Disk I/O Latency)
* **Finding:** Operations like updating macro steps or saving application state execute individual `INSERT`/`UPDATE` statements without explicitly wrapping batch edits in a `using var transaction = connection.BeginTransaction()`. Furthermore, SQLite `journal_mode` defaults to DELETE instead of WAL (Write-Ahead Logging).
* **Remediation & Best Practice:**
  1. Enable WAL mode and `PRAGMA synchronous = NORMAL;` to allow non-blocking concurrent reads during macro execution.
  2. Add `PRAGMA busy_timeout = 5000;` to eliminate "Database is locked" concurrency crashes.
  3. Wrap batch macro saves, step reorders, and bulk operations in explicit SQLite transactions.
* **Reference:** [Microsoft.Data.Sqlite Performance Best Practices](https://learn.microsoft.com/en-us/dotnet/standard/data/sqlite/async)

### 3.2 Reflection JsonSerializer Bottleneck
* **Location:** `MacroItem` (2300+ lines model), `AppConfig`, `MacroDatabase`
* **Risk:** Low/Medium (CPU & Startup Latency)
* **Finding:** Heavy reliance on reflection-based `System.Text.Json.JsonSerializer` for deep object trees (`MacroStep` with 100+ properties).
* **Remediation:** Implement `.NET C# Source Generators` (`JsonSerializerContext`) to eliminate runtime reflection during macro load/save cycles.

---

## 4. 🏗️ Architectural & Maintainability Debt

### 4.1 Monolithic God Classes
* **`RemoteServerService.cs` (5607 lines):** Mixes networking (HttpListener), Security (PIN auth), HTML rendering (Controller UI), Soundboard file storage, and Windows Volume Win32 P/Invoke calls.
* **`ScriptCompilerService.cs` (5264 lines):** Single file containing code generation for 30+ action block types.
* **`MacroItem.cs` (2261 lines):** Combines data model, step execution flags, UI binding logic, and step property metadata.

### 4.2 Lack of Dependency Injection & Testability
* Services instantiate dependencies directly via `new` or rely on static singletons (`Managers.ScriptManager.Stop()`, `ConfigManager.Current`).
* Unit testing is currently impossible without executing real Win32 APIs or running AHK processes.

---

## 📊 Summary Matrix & Prioritization Recommendations

| Ref | Area | Priority | Effort | Impact |
|-----|------|----------|--------|--------|
| **S-01** | Fix `/api/soundboard/upload` missing auth check | 🔴 Critical | 🟢 5 mins | Eliminates unauthorized LAN file writes |
| **P-01** | Replace `async void` in ViewModels with `async Task` | 🔴 Critical | 🟡 1 hour | Prevents silent WPF application crashes |
| **S-02** | Support multi-device session tokens (`ConcurrentDictionary`) | 🟡 High | 🟡 30 mins | Fixes mobile remote multi-device dropouts |
| **D-01** | Wrap SQLite batch operations in explicit transactions | 🟡 High | 🟡 1 hour | 10x-50x faster macro database saves |
| **A-01** | Split `RemoteServerService.cs` into modular API controllers | 🟡 Medium | 🔴 3 hours | Drastically improves code maintainability |

---

## 🔗 Related Documents

* [[block-qa-audit]] — Block-level QA audit
* [[packaging]] — Pre-packaging checklist
