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
