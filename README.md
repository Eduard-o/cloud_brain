# cloud_brain

Personal knowledge base / working memory for Claude Code, backed up to GitHub.

## Structure
- `CLAUDE.md` — auto-loaded working memory, kept short and current
- `memory/work/` — work-related context (roles, companies, colleagues)
- `memory/personal/` — personal context (self, life, relationships)
- `memory/projects/` — one file per individual project we work on
- `memory/skills/` — Claude Code skills meant to be available globally (not scoped to one project's own `.claude/skills/`)
- `memory/glossary.md` — full decoder ring (terms, people, codenames)
- `memory/decisions.md` — append-only decisions log

## Usage
1. Claude Code reads `CLAUDE.md` at session start
2. Relevant `memory/*.md` files get pulled in as needed
3. Update files as things change; commit + push to keep GitHub in sync
