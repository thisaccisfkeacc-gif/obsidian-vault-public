# 🤖 Agent Round-Robin — Watchdog Improvement prompt

Copy the prompt below and paste it to other agents. Share their answers back and we'll evaluate them.

---

## Prompt to paste:

```
I need your opinion on optimizing an Android anti-tamper watchdog.

Background:
- We build a focus/content-blocking Android app (Kotlin, Jetpack Compose).
- When the user enables "Master Lock", settings freeze and anti-uninstall is active.
- A foreground service called GuardianService protects the app. Its job: make sure the accessibility service can't be turned off by the user (that would let them bypass the blocker).
- CURRENT approach: GuardianService checks whether accessibility is still enabled, using a Handler.postDelayed loop every 2 seconds (CHECK_INTERVAL_MS = 2000L). If disabled, it shows a full-screen "brick" overlay that can't be dismissed.

Question 1: Is polling every 2 seconds a battery drain, or is it negligible?
Question 2: Is polling every 2sec the right design, or is there a smarter, less wasteful way?
Question 3: Suggest concrete alternatives to constant polling. I've heard of "listening for change events" (e.g. Android broadcasting when an app is disabled/uninstalled, or using PackageManager receivers / ACTION_PACKAGE_CHANGED / PACKAGE_REMOVED / accessibility state signals). Evaluate that idea: is it reliable enough for a tamper-proof watchdog? What about restoration time (poll catches instantly, events may lag)?
Question 4: Give your recommended design (what interval, what events to listen for, any best practice) for this anti-tamper use case specifically — balancing battery, reliability, and how quickly the overlay must appear when the user tries to disable accessibility.
Question 5: Any alternative architectures (e.g. using WorkManager periodic, alarms, watchdog placed OUTSIDE the app with another app/companion) that are stronger for tamper resistance?

Keep it concise and practical. I want plain, implementation-ready suggestions — not theory.