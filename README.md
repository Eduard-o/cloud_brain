# cloud_brain

Personal knowledge base / working memory for Claude Code, backed up to GitHub.

## Structure
- `CLAUDE.md` — auto-loaded working memory, kept short and current
- `memory/work/` — work-related context (e.g. luulive.md)
- `memory/personal/` — personal context (e.g. relocation.md)
- `memory/decisions.md` — append-only decisions log

## Usage
1. Claude Code reads `CLAUDE.md` at session start
2. Relevant `memory/*.md` files get pulled in as needed
3. Update files as things change; commit + push to keep GitHub in sync
