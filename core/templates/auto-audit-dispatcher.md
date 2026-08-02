# 🤖 Auto Audit Dispatcher

> **This is a self-directing prompt. Read it fully before doing anything.**
> You will pick your own task, run the full audit, and update the checklist when done.
> No human will give you the next step — you figure it out yourself.

---

## Step 1 — Find Your Task

Open this file and read it completely:
`C:\Users\Maaz\Documents\New folder\Obsidian Vault\audit-areas.md`

Find the **first area** with status `⬜ Not Started` — that is your assigned task.
Change its status from `⬜ Not Started` to `🔄 In Progress` immediately so no other agent picks the same task.

---

## Step 2 — Read the Rules

Before doing anything else, read these files in order:
1. `C:\Users\Maaz\Documents\New folder\Obsidian Vault\core\SCHEMA.md`
2. `C:\Users\Maaz\Documents\New folder\Obsidian Vault\core\wiki\index.md`
3. `C:\Users\Maaz\Documents\New folder\Obsidian Vault\core\GOTCHAS.md` — read every gotcha, especially #15 (git safety)
4. `C:\Users\Maaz\Documents\New folder\Obsidian Vault\core\templates\goal-audit-prompt.md` — this is the audit methodology you must follow

---

## Step 3 — Run the Full Audit

Using the methodology from `goal-audit-prompt.md`, run the full 5-phase audit on your assigned area:

- **Phase 1:** Prep — read wiki docs relevant to your area, build a file checklist in `<zone_name>_audit.md`
- **Phase 2:** Scan Loop — scan every file, log all findings, tick off each file
- **Phase 3:** Filter — 3-pass filter (obvious false flags → stress test → confidence rating)
- **Phase 4:** Fix — fix all `[NEEDS FIX]` items, build after every change
- **Phase 5:** Wrap up — write summary, update wiki/log.md, update audit-areas.md

---

## Step 4 — Strict Rules (Non-Negotiable)

🚫 **GIT:** NEVER run git revert, git restore, git reset, or git checkout. Manually undo file edits only.

🔒 **SCOPE:** Only touch files that belong to your assigned area. Nothing else.

🐛 **OUT-OF-SCOPE BUGS:** If you find a bug outside your area, log it under `🐛 Other Bugs Found` in your audit file. Do NOT fix it.

⚠️ **SMART MENU:** Never touch, scan, or modify the Smart Menu / Clipboard Manager code under any circumstances.

🔨 **BUILD:** Run `dotnet build "PowerX_Keys_V2.csproj"` from:
`C:\Users\Maaz\Documents\New folder\PowerX Keys\PowerX_Keys_V2_Rebuild\PowerX_Keys_V2\`
after every single file change. If build fails twice on the same fix, revert it manually and mark `[SKIPPED — BUILD FAILED]`.

---

## Step 5 — Mark Done

When your full audit is complete, open `audit-areas.md` and change your area's status from `🔄 In Progress` to `✅ Done`.

That's it. You're done. 🎯
