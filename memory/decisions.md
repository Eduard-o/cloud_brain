# Decisions Log

> Append-only. Newest at top. Never rewrite past entries — if a decision changes, add a new entry noting the change.

## Format
`### YYYY-MM-DD — [short title]`
- Decision:
- Why:
- Status: (active / superseded / reversed)

---

<!-- Add entries below this line -->

### 2026-07-16 — Bash permission wildcard syntax and settings file sync direction
- Decision: Bash permission rules in `.claude/settings*.json` use `command:*` (colon) syntax, not `command *` (space) — the latter doesn't reliably match real invocations. `settings.local.json` (gitignored, not pushed to GitHub) is the source of truth for permissions; `settings.json` (tracked, pushed) should be kept in sync to mirror it.
- Why: A manual edit changed several rules from `:*` to ` *` syntax, which likely caused permission prompts to reappear despite allow rules being present. Also found and fixed an invalid trailing comma in `settings.local.json` that would have broken JSON parsing.
- Status: active

### 2026-07-16 — memory/skills/ requires a junction to actually load globally
- Decision: `memory/skills/` in this repo holds version-controlled skill files, but Claude Code does NOT scan that path directly. It only auto-discovers skills from `~/.claude/skills/<name>/SKILL.md` (global) or a project's own `.claude/skills/<name>/SKILL.md` (local). To make a skill in `memory/skills/` actually load globally, create a directory junction from `~/.claude/skills/<name>` to `memory/skills/<name>` (Windows: `mklink /J "C:\Users\<user>\.claude\skills\<name>" "D:\Software Projects\cloud_brain\memory\skills\<name>"`). Junctions don't require admin rights, unlike symbolic links.
- Why: Verified via research — added `gdunit4-test-writer` to `memory/skills/` first, confirmed it did NOT appear in Claude Code's available-skills list until the junction was created, then confirmed it did appear immediately after.
- Status: active
