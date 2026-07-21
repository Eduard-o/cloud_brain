# Decisions Log

> Append-only. Newest at top. Never rewrite past entries — if a decision changes, add a new entry noting the change.

## Format
`### YYYY-MM-DD — [short title]`
- Decision:
- Why:
- Status: (active / superseded / reversed)

---

<!-- Add entries below this line -->

### 2026-07-20 — KaetramDB git-push CD block resolved
- Decision: N/A (status update, not a new decision).
- Why: User fixed the Netlify account/team setting that was blocking git-push-triggered builds (see 2026-07-20 "deployed live" entry below). Confirmed via a real push-triggered deploy reaching `ready` state and the live site reflecting it. Manual build-hook triggering after each push is no longer necessary.
- Status: resolves the "unresolved" status noted in the entry below. See memory/projects/project-kaetramdb.md § Deployment.

### 2026-07-20 — KaetramDB deployed live; git-push CD blocked, using build hook instead
- Decision: Deployed to Netlify (private GitHub repo `Eduard-o/kaetramdb`, site `kaetramdb.netlify.app`, site ID `afee928f-70b4-43e6-9685-226f57f5c6df`). Git-based continuous deployment is connected, but actual `git push` builds are blocked by Netlify with "Unrecognized Git contributor — This plan allows only verified account members to push to private repos," even for commits authored by the repo owner. Getting a push's changes live currently requires manually POSTing to the site's build hook after each push.
- Why: This is a Netlify account/team-level restriction, not something resolvable via CLI, `netlify.toml`, or any workaround I should apply — per the netlify-deploy skill, a broken documented happy path gets surfaced to the user, not routed around. The build-hook trigger is a legitimate alternate path (same one the weekly-rebuild scheduled function already uses), not a hack.
- Status: active, unresolved. User needs to check Netlify Team/Account member settings to fix push-to-deploy. See memory/projects/project-kaetramdb.md § Deployment.

### 2026-07-19 — KaetramDB search UI: hand-built instead of Pagefind's Component UI
- Decision: Replaced Pagefind's stock `<Search>` widget (from `astro-pagefind/components`) with a custom hover-reveal dropdown built directly on Pagefind's raw JS search API. Kept the `pagefind()` Astro integration for build-time indexing only.
- Why: The stock widget's CSS uses `--pf-*` variables scoped under `[data-pf-theme="dark"]` (a different, newer scheme than the `--pagefind-ui-*` variables originally assumed), and its result cards always render a title+excerpt — neither fit the "dark theme" + "icon and name only, no extra data" requirements. Full custom markup was simpler than fighting the prebuilt component's opinionated styling.
- Status: active. See memory/projects/project-kaetramdb.md.

### 2026-07-19 — KaetramDB v1 stack and scope
- Decision: New project "KaetramDB" — unofficial Kaetram item/mob database with search, styled after BitJita's dark theme. Astro (static) + Tailwind + Pagefind search + Netlify hosting, data pulled from the Kaetram public API at build time (no runtime backend). Weekly build hook refresh, not daily. v1 scope is items + mobs only; Exchange market data and leaderboards deferred.
- Why: Kaetram has no public live-build GitHub repo, so the API is the freshest data source; item/mob data changes infrequently (only on game patches) so a static site with periodic rebuilds is simpler and cheaper than a live backend.
- Status: active. See memory/projects/project-kaetramdb.md.

### 2026-07-16 — Bash permission wildcard syntax and settings file sync direction
- Decision: Bash permission rules in `.claude/settings*.json` use `command:*` (colon) syntax, not `command *` (space) — the latter doesn't reliably match real invocations. `settings.local.json` (gitignored, not pushed to GitHub) is the source of truth for permissions; `settings.json` (tracked, pushed) should be kept in sync to mirror it.
- Why: A manual edit changed several rules from `:*` to ` *` syntax, which likely caused permission prompts to reappear despite allow rules being present. Also found and fixed an invalid trailing comma in `settings.local.json` that would have broken JSON parsing.
- Status: active

### 2026-07-16 — memory/skills/ requires a junction to actually load globally
- Decision: `memory/skills/` in this repo holds version-controlled skill files, but Claude Code does NOT scan that path directly. It only auto-discovers skills from `~/.claude/skills/<name>/SKILL.md` (global) or a project's own `.claude/skills/<name>/SKILL.md` (local). To make a skill in `memory/skills/` actually load globally, create a directory junction from `~/.claude/skills/<name>` to `memory/skills/<name>` (Windows: `mklink /J "C:\Users\<user>\.claude\skills\<name>" "D:\Software Projects\cloud_brain\memory\skills\<name>"`). Junctions don't require admin rights, unlike symbolic links.
- Why: Verified via research — added `gdunit4-test-writer` to `memory/skills/` first, confirmed it did NOT appear in Claude Code's available-skills list until the junction was created, then confirmed it did appear immediately after.
- Status: active
