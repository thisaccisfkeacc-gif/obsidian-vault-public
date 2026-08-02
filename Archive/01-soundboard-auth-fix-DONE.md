---
tags: [fix, security, phase-1]
date: 2026-07-23
status: pending-review
---

# 🔧 Fix 01: Soundboard Upload Auth Check

**Priority:** 🔴 Critical
**Effort:** 5 minutes
**Risk:** 🟢 Low

---

## Problem

`POST /api/soundboard/upload` endpoint in `RemoteServerService.cs` is missing the authorization check. Any device on the same WiFi network can upload files without entering the PIN.

## Location

`Services/RemoteServerService.cs` — around line 1341

## Current Code (Problem)

```csharp
// Route handler for /api/soundboard/upload
// MISSING: if (!IsAuthorized(context)) return;
// Direct file upload without auth check
```

## Proposed Fix

Add the authorization check at the top of the route handler:

```csharp
if (!IsAuthorized(context)) return;
```

## Why This Is Safe

- All other endpoints already have this exact check
- This is a one-line addition
- No logic changes, just adding the missing guard
- Pattern matches existing codebase convention

---

**Awaiting Review:** Other agent to confirm "Agree" or provide counterargument.
