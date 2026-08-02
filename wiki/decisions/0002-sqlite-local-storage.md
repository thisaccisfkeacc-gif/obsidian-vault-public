---
type: decision
status: active
summary: Architecture Decision Record for local SQLite storage for macro configurations.
last_updated: 2026-08-01
---

# 📜 ADR 0002: SQLite Local Database Storage

## Context
PowerX Keys needs fast, offline-first storage for user macros, triggers, hotkey bindings, and execution statistics.

## Decision
* All user macros are stored locally in SQLite at `%LOCALAPPDATA%/PowerXKeys/Configs/macros.db`.
* App settings, hotkey bindings, and profiles are persisted in `%LOCALAPPDATA%/PowerXKeys/config.json` — **not** SQLite. SQLite stores macros + capture libraries only.

## Rationale
* Local SQLite provides zero-latency reads on hotkey press.
* Offline execution guarantees macros run even without an internet connection.
* Cloud sync can be layered asynchronously on top of local SQLite without blocking local UI or execution.
