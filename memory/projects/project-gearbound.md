# Project Gearbound

**Type:** personal
**Status:** active (scoped, prototype not yet started)
**Last updated:** 2026-07-16
**Repo:** TBD
**Local path:** TBD

## Overview
- Godot game dev idea, still in early concept/discussion phase — not yet scoped or named for real.
- Genre direction: bullet-hell MMO combat (like Realm of the Mad God) combined with open, gear-defined class-free progression (like Albion Online), rather than class-locked progression.
- Niche being targeted: most bullet-hell games are class-locked, most open-progression MMOs are slower/tab-target combat — combining fast bullet-hell action with open gear-based builds is unusual.

## Key people
- Eduardo (solo, at least for now)

## Agreements / important facts
- Reference points so far: Realm of the Mad God (bullet-hell, permadeath, retro pixel art, class-based) and Albion Online (class-free, gear-defines-role, economy-heavy).
- Eduardo is explicitly not interested in class-locked progression.
- **Permadeath: confirmed yes.**
- **No character level (for now):** all characters start with identical base stats. Power comes solely from player skill, equipped gear, and mastery-track bonuses — no XP-based leveling layered on top.
- **Gear system:** gear determines skills and stat bonuses (not raw combat mechanics), giving players build freedom without changing the bullet-hell feel. Plans to add an Albion-style gear mastery tree for long-term progression.
- **Mastery tree (basic, prototype version):** per-weapon-type mastery track — using a weapon/gadget type earns mastery XP toward that specific type, unlocking a small number of flat nodes (~5-8) with minor passive bonuses (e.g. damage, fire rate, reload) relevant to that type. Linear unlock line, no branching choices yet. Chosen to be small deliberately since it's character-bound and wiped on permadeath — losing a big investment on death would be too punishing for repeated playtest sessions.
- **Gear slots (6):** Weapon (active, primary auto-attack), Gadget (active, cooldown-based skill — grenades/EMP/turrets/medkits/traps, post-apoc reskin of RotMG's "ability" slot), Chest (passive), Feet (passive), Head (passive), Weapon Mod (passive).
- **Rarity system:** gear has rarity tiers; higher rarity = more/stronger buffs on the item.
- **Combat controls:** twin-stick style (WASD movement + mouse aim), independent of facing/movement direction.
- **Dodge/evasion:** no dedicated dash/dodge mechanic — pure movement/positioning (RotMG-style), consistent with permadeath stakes.
- **Enemy design:** mutated creatures (radiation/mutation-twisted), fitting the post-apoc theme.
- **Difficulty structure:** zone-based tiers in the open world, with randomly appearing event bosses in mid/high-tier zones. Dungeons are tiered (Tier 1, Tier 2, etc.) rather than difficulty-ramping room-by-room — each tier is **procedurally generated** (rule-based, not a fixed layout), with higher tiers producing more rooms, harder enemies, and better gear drop chances. Traps are a planned future addition to dungeon generation, not in the initial prototype scope.
- **World structure:** Albion-style hub cities. Difficulty expands outward from cities, with a central high-level city gated behind having to cross the highest-difficulty zone to reach it.
- **Cities are safe zones** (Albion-style) — no combat/PvP allowed in cities.
- **No player trading system in the prototype.** Players can informally trade by dropping items on the ground for others to pick up — no dedicated trade UI/market/auction house yet.
- **PvP: friendly-fire safe (for now).** Outside cities, players cannot damage each other — combat danger comes only from enemies, not other players.
- **Death/respawn flow:** on permadeath, the corpse is visible to other nearby players but not lootable (for now). The player is sent to a character selection screen, where they can create a new character or pick an existing one from an available character slot — implies accounts support multiple character slots.
- **Progression persistence:** mastery-tree progress is character-bound (wiped on permadeath, full RotMG-style stakes). Achievements and cosmetics are account-bound (survive death, purely for long-term player retention/pride, no power).
- **Two-phase plan:** ship a small prototype first to prove the concept and playtest with friends, then a follow-on "1.0" release with fuller MMO scope.
- **Prototype scope (phase 1):**
  - Character creation
  - Small mastery tree (no gear crafting — gear is drop-only, from enemies and dungeon chests)
  - A few basic achievements (account-bound)
  - One world with a handful of enemies at varying difficulty
  - One dungeon
  - Basic multiplayer — a few friends on the same shared instance (no full MMO server infra/matchmaking/accounts yet), specifically to playtest whether the game is fun with others before investing in full MMO infrastructure
- **Theme/setting: post-apocalyptic.** Chosen over fantasy specifically for differentiation — fantasy bullet-hell MMOs (RotMG, etc.) are the crowded default; post-apoc bullet-hell combined with class-free MMO progression has no obvious direct competitor. Tradeoff to watch: fewer ready-made post-apoc asset packs exist compared to fantasy.
- **Art style: stylized low-poly 3D, fixed top-down/isometric camera** (not free-roaming). Fixed camera preserves bullet-hell projectile readability despite being 3D. Low-poly style chosen to lean on asset packs (e.g. Synty Studios, Kenney) given Eduardo has little art skill currently, while still being genuine 3D practice toward his longer-term interest in learning Blender.
- **Distribution:** eventually release on Steam (not just standalone) — plan to use GodotSteam for Steamworks integration (cloud saves, achievements, lobbies).
- **Hosting/infra:** dedicated server only, no peer-to-peer. Eduardo has Podman available locally.
  - Game server: Godot headless/server export, containerized (Podman), runs on Eduardo's own machine for the prototype/playtest phase.
  - Character data storage: Postgres, also containerized locally alongside the game server for now — since the game server already ties multiplayer uptime to Eduardo's machine, a local DB adds no new single point of failure and avoids free-tier hosted-DB limits (cold starts, egress caps, connection management).
  - Considered hosted Postgres (Neon, Supabase — Supabase already available as an MCP connector in this environment) for the DB; concluded hosted makes more sense once the game server itself moves off Eduardo's machine (VPS or full-MMO phase), not before. Migration path is simple either way (plain Postgres, no lock-in — `pg_dump`/restore).

## Open questions / next steps
- Real project name/title.
- Phase 2 ("1.0") scope: what full MMO infrastructure/features get added after the prototype proves out?
- Phase 2 idea — public status website + API: a public site showing game server status (online/down indicator, player count), later expanded into an API for player stat lookups, live server conditions, and a wiki (e.g. enemy drop tables). Explicitly deferred past the prototype. Still needs a decision on how a public site reaches the locally-hosted game server's live state (e.g. a tunnel like Cloudflare Tunnel/ngrok) once revisited.

## Notes
- "Gearbound" is a placeholder name, not a final title.
