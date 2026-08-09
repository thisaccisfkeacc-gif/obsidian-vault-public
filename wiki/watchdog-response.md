# 🛡️ Watchdog Optimization & Anti-Tamper Analysis

> **Linked Prompt:** [[watchdog-prompt]]  
> **Target Application:** Lockin (Kotlin, Jetpack Compose)  
> **Component:** `GuardianService` / Accessibility Watchdog  

---

## ⚡ Quick Summary (Non-Technical & Simple)

### 1. Does 2-second polling drain battery?
**Yes.** Checking every 2 seconds keeps the phone's CPU awake 24/7. Android and OEM battery managers (Samsung, Xiaomi) will detect this and forcibly kill your app.

### 2. Is 2-second polling necessary?
**No.** Users can only turn off accessibility when actively using the **Settings app**. Checking every 2 seconds while the screen is OFF or while the user is watching videos is 99% wasted work.

### 3. Are "change listeners" fast and reliable?
**Yes!** Instead of asking *"Is accessibility off yet?"* every 2 seconds, we tell Android *"Notify me immediately if accessibility changes!"*. It responds in milliseconds (< 50ms) and uses 0% extra battery while waiting.

### 4. What is the recommended design?
A **Smart Hybrid Approach**:
* **Instant Alarm:** Use `ContentObserver` so the block overlay pops up in < 50ms when accessibility is toggled off.
* **Smart Polling:** Turn off polling when the screen is dark/locked. Only check frequently if the user actually opens the Settings app.

### 5. Stronger Anti-Tamper Methods
* **Pre-emptive Blocking:** Detect when the user opens `com.android.settings` and send them back to the Home screen *before* they tap the disable toggle.
* **Device Admin Lock:** Use `dpm.setUninstallBlocked()` to freeze the uninstall button natively via Android system policy.
* **Buddy Guard (Dual Process):** Run a lightweight `:watchdog` companion process that instantly revives `GuardianService` if the main process is force-stopped.

---

## 📐 Deep Technical Implementation Guide

### Question 1: Battery Impact Analysis
* **Mechanism:** `Handler.postDelayed(CHECK_INTERVAL_MS = 2000L)` in `GuardianService`.
* **Drawbacks:**
  1. Prevents CPU from entering deep-sleep C-states (violating Doze mode principles).
  2. Causes 43,200 Binder IPC calls per day to `system_server` (`Settings.Secure`).
  3. Triggers OEM background restrictions (MIUI/HyperOS, Samsung Device Care) which terminate `START_STICKY` services flagged as battery hogs.

### Question 2: Polling Design Assessment
* **Verdict:** Flawed for 24/7 background execution.
* **Context Scope:** Tampering requires active user interaction inside `com.android.settings`. Polling when `PowerManager.isInteractive == false` or when the active window is non-settings yields zero security benefit.

### Question 3: Event Listeners vs. Polling

```
User toggles Off Accessibility in Settings
       │
       ├──> [ContentObserver.onChange()] ──────────> Brick Overlay Shown (< 50ms)
       │
       └──> [Handler postDelayed (2s loop)] ───────> Brick Overlay Shown (0 - 2000ms delay)
```

1. **`ContentObserver` on `Settings.Secure`**:
   ```kotlin
   val uri = Settings.Secure.getUriFor(Settings.Secure.ENABLED_ACCESSIBILITY_SERVICES)
   contentResolver.registerContentObserver(uri, false, object : ContentObserver(Handler(Looper.getMainLooper())) {
       override fun onChange(selfChange: Boolean) {
           checkAccessibilityAndEnforce()
       }
   })
   ```
2. **`AccessibilityStateChangeListener`**:
   ```kotlin
   val am = getSystemService(Context.ACCESSIBILITY_SERVICE) as AccessibilityManager
   am.addAccessibilityStateChangeListener { enabled ->
       if (!enabled && isMasterLockActive()) {
           showBrickOverlay()
       }
   }
   ```
* **Performance:** Instant execution (< 50ms latency), 0% idle CPU wakeups.

---

### Question 4: Recommended Architecture

```
+-------------------------------------------------------------------------------+
|                             GuardianService                                   |
|                                                                               |
|  [Event Driver]  ContentObserver / AccessibilityStateChangeListener           |
|                  --> Instant Overlay Enforcement (< 50ms)                     |
|                                                                               |
|  [Adaptive Loop] Screen ON & User in Settings  --> Poll @ 500ms               |
|                  Screen ON & Normal App Use    --> Heartbeat @ 30s            |
|                  Screen OFF                    --> Pause Loop (0% Battery)    |
+-------------------------------------------------------------------------------+
```

1. **Primary Enforcer:** Event-driven `ContentObserver` for instant response.
2. **Adaptive Polling State Machine:**
   * **Screen OFF (`ACTION_SCREEN_OFF`):** Unregister polling handlers completely.
   * **User in Settings (`com.android.settings`):** Enable aggressive 500ms check loop.
   * **Normal Phone Usage:** Relax check interval to 30–60 seconds as safety net.
3. **Pre-emptive Accessibility Intercept:**
   Inspect `TYPE_WINDOW_STATE_CHANGED` events inside the app's `AccessibilityService`. If `packageName == "com.android.settings"` and text contains target service name, execute `GLOBAL_ACTION_HOME` or launch overlay pre-emptively.

---

### Question 5: Advanced Anti-Tamper Strategies

1. **Native Uninstall Lock via Device Policy Manager:**
   ```kotlin
   val dpm = getSystemService(Context.DEVICE_POLICY_SERVICE) as DevicePolicyManager
   val compName = ComponentName(context, AdminReceiver::class.java)
   if (dpm.isAdminActive(compName)) {
       dpm.setUninstallBlocked(compName, packageName, true)
   }
   ```
2. **Dual-Process Mutual Resurrection (`:watchdog` process):**
   * Define `:watchdog` process in `AndroidManifest.xml`.
   * Bind via local socket or `IBinder.DeathRecipient`.
   * If user triggers "Force Stop" on main app, `:watchdog` gets instant `binderDied()` callback and immediately launches `GuardianService` overlay.
3. **`AlarmManager` Safety Net:**
   * Schedule `AlarmManager.setExactAndAllowWhileIdle()` every 15 minutes to guarantee re-wake even if system purges process during deep sleep.
