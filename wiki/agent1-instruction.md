You are the DOCUMENTATION BUILDER for the "Lockin" (App Block V2) Android project.

Your job: create ONE single, self-contained reference document on the DESKTOP that gives a brand-new agent (Agent 2) full context about the project so they can pick up and continue work without any prior conversation history.

## STEP A — Read these source files first
Read every file below, in order, to gather accurate facts. Do not invent anything.

1. `c:\Users\Maaz\Documents\New folder\Obsidian Vault\App_Block.md` — the master PRD
2. `c:\Users\Maaz\Documents\New folder\Obsidian Vault\wiki\build-queue.md` — the task list + decisions
3. `c:\Users\Maaz\Documents\New folder\Obsidian Vault\wiki\implementation-handoff.md` — the build spec (all 7 steps + details)
4. `c:\Users\Maaz\Documents\New folder\Obsidian Vault\wiki\log.md` — the audit report
5. `c:\Users\Maaz\Documents\New folder\Obsidian Vault\wiki\watchdog-prompt.md` — watchdog question asked to agents
6. `c:\Users\Maaz\Documents\New folder\Obsidian Vault\wiki\watchdog-response.md` — the watchdog suggestions from an external agent

## STEP B — Create the master document
Create a file called:
`C:\Users\Daniel\Documents\LOCKIN_MASTER_DOCUMENT.md`

Format it as a clean, fully organized, reference document. It MUST include these sections, verbatim facts from the sources:

### 1. Project Overview
- What Lockin is (anti-bypass Android productivity/focus app: blocks adult content, social media, distractions)
- Tech stack: native Kotlin + Jetpack Compose ONLY. NO Flutter anywhere (explicit decision to avoid double build time).
- Architecture: UI screens in `screens/`, backend as Services + Singleton managers, native Kotlin.
- App project path: `C:\Users\Daniel\Projects\Lockin\` (Reef codebase)
- The HTML `PowerX_Block_UI_Preview` is visual inspiration ONLY — never port it.

### 2. The Golden Pipeline / How blocking works
- AccessibilityService reads browser URL bars + detects app windows, then blocks/redirects/kicks.
- Master Lock freezes all settings; unlock options replace USB/ADB.

### 3. Feature Status Summary (from log.md + build-queue)
List EACH feature with a status of: ✅ DONE / 🔴 NOT BUILT / 🟡 DESIGNED:
- Web & keyword filter — done (101,143 adult domains, 246 keywords, fail-closed browsers 4 readable + 21 blocked)
- App blocking (time limits, floating timer, whitelist) — done
- Master Lock FREEZE + unlock backend — done (timer lock + escalating gibberish 64→256, ADB restricted to DEBUG only)
- Anti-tamper guardian — done (watchdog + Device Admin + brick overlay)
- Shorts/Reels blocker — done: targeted, YT Shorts + IG Reels, independent toggles
- Build queue pending tasks — list 1→7 from build-queue.md with their specs & decisions (escalation rules, 1hr/week credit no-carryover, reverse cooldown + 3-min gate, "timer cooldown" wording, no gamification)
- Known gaps flagged in the code audit (gibberish not wired initially → later fixed; timed pass gold)

### 4. Anti-Tamper Defense Ladder (IMPORTANT — the agent asked for this)
Lay out clearly, layers from weakest to strongest:
- Plain Device Admin (no reset) — user must deactivate admin first; accessibility detects your Settings/App-Info page and bounces the user out; bypassable ~10s if determined.
- Work Profile / Profile Owner — real policy control but only inside work profile, too limiting.
- Device OWNER via `adb shell dpm set-device-owner` — needs a factory-fresh device with no accounts; uninstall-block, no Force Stop, pinned accessibility service, no battery cost. Requires reset to set up and reset to undo.
State the current decision: current phone = use existing Admin + accessibility tamper protection. Don't reset a daily-use phone. Device Owner planned for a dedicated/second fresh device later.

### 5. Watchdog Architecture (approved design)
From The handoff STEP 6 + the watchdog-response:
- Primary: ContentObserver on `ENABLED_ACCESSIBILITY_SERVICES` + `AccessibilityStateChangeListener` (instant, <50ms).
- Adaptive: screen ON + in Settings = 500ms poll; screen ON + normal = 30–60s heartbeat; screen OFF = pause.
- Do NOT block the whole Settings app — only interfere when user tries to disable OUR accessibility service/Force Stop/our App Info page.
- Optional future Device Owner makes it mostly never needed.

### 6. Decisions & Design Notes Log
Collect ALL explicit decisions as bullet entries with "(decision)" tag:
- No Flutter
- 64 / +10 per wrong / max 256, remember last-used on new session
- Timer unlock with warning popup
- Credit: 1 hr / WEEK, no carryover, one spend per session (session = 1-hr block window refresh)
- Reverse cooldown: gradual refill, 3-min away gate, WATCH word "timer cooldown", NOT battery wording
- Pass presets 5,10,15,20,1h on block screen; COMPACT ICON-BUTTON top-right, not attention-seeking; hidden during Master Lock
- Quick block session presets 15,30,60,120spent; adult toggle works
- No "Forever" option, no streak bonus
- Gamification archived behind ENABLE_GAMIFICATION=false

### 7. Doc Index (for easy nav)
List of wiki internal documents and what each is for.

## STEP C — Quality rules
- It must be ONE single file, complete enough that Agent 2 needs no other files and no chat history.
- Use tables where practical (feature status, decision log).
- Plain English, friendly bullet points / emoji. No raw code unless it's insightful.
- State facts only. If something is unknown or a brand-new gap, mark it "⚠️ TO CONFIRM".
- End with a short "Next Steps / What to build next" section (queue order).

Return at the end: the exact file path of the created document + a 5-bullet summary of what it contains.