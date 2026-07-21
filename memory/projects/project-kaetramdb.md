# KaetramDB

**Type:** personal
**Status:** active, deployed and live
**Repo:** https://github.com/Eduard-o/kaetramdb (private)
**Live site:** https://kaetramdb.netlify.app
**Local path:** D:\Software Projects\kaetramdb

## Overview
Unofficial third-party item/mob database for [Kaetram](https://kaetram.com) (2D pixel-art
MMORPG), in the spirit of BitJita (BitCraft) and FareverDB (Farever). Inspired by BitJita's
black/dark theme. Built on Kaetram's public API (https://kaetram.com/api-docs) since Kaetram
doesn't have a public live-build GitHub repo, so the API is the most up-to-date data source.

v1 scope: item + mob database with search. Crafting recipes and a live Exchange (market)
page were added after v1. Leaderboards remain deferred.

## Stack
- Astro (static site) + Tailwind CSS, dark/black theme
- Pagefind (via `astro-pagefind`'s integration, for build-time indexing only) — but the
  search **UI** is hand-built (`src/components/HeaderSearch.astro`), not Pagefind's stock
  Component UI. The stock UI uses `--pf-*` CSS variables (not the older `--pagefind-ui-*`
  ones) and always renders title+excerpt cards, which didn't fit the dark theme / icon+name
  requirement — see decisions log 2026-07-19.
- Netlify for hosting; `netlify.toml` builds via `npm run build` and publishes `dist`
- No runtime backend — `scripts/fetch-kaetram-data.mjs` pulls item/mob/drop-table/skills
  dumps from `api.kaetram.com` at build time into `src/data/*.json`
- Item icons (1,834, ~2.2MB total) are downloaded into `public/icons/*.png` at fetch time
  and served locally rather than hotlinked from the Kaetram API — avoids depending on their
  CDN at request time and is friendlier to their API. Missing icons (and all mob icons —
  Kaetram's API has no mob-icon endpoint) fall back to a first-letter placeholder
  (`src/components/IconFallback.astro`, used by `ItemIcon.astro` and the mobs pages).
- Items/mobs listings are unpaginated (all items/mobs rendered at once) with a client-side
  sort `<select>` that reorders DOM nodes — needed because sorting has to work across the
  whole dataset, and Astro's static `paginate()` can't respond to a runtime-selected sort.

## Agreements / important facts
- Data refresh cadence is **weekly**, not daily — item/mob data only changes when the game
  patches. A Netlify scheduled function (`netlify/functions/weekly-rebuild.mts`, Mondays
  06:00 UTC) POSTs to a Netlify build hook to trigger the rebuild. Requires a `BUILD_HOOK_URL`
  env var to be set on the Netlify site (not yet done — site doesn't exist yet).
- Kaetram API quirk: `/api/v1/items/count` reports 3,589 but the actual `/api/v1/items` dump
  only contains ~1,834 unique keys. Went with what `/items` actually returns since that's
  what's queryable (search/detail endpoints use the same underlying data).
- Kaetram mob records are sparse and inconsistent — most only have `name`, `drops`,
  `animation`, and sometimes `boss: true`. No reliable level/HP/combat-stat fields across the
  board, so the mob detail page renders whatever fields exist generically rather than
  assuming a fixed schema.
- Item `requirements.skill`/`secondSkill` are ordinal indexes into `/api/v1/skills`
  (e.g. `7` → `DEFENSE`), not free-standing IDs — `getSkillName()` in `src/lib/kaetram.ts`
  maps them to title-cased names for display.
- Dynamic `import()` of a build-generated, non-module-graph path (`/pagefind/pagefind.js`)
  breaks under Vite even with `/* @vite-ignore */` — it still wraps the import with preload
  helpers that throw `__VITE_PRELOAD__ is not defined` at runtime in the production bundle.
  Fix: route the import through `new Function("path", "return import(path)")` so Vite's
  static analyzer never sees the specifier at all.

## Crafting recipes (added 2026-07-19)
- Item pages now show a "Crafting" section (skill, level, XP, success chance, ingredients
  with icons linking to their item pages, and fail-result for chiseling recipes) for any
  item that has a recipe.
- Kaetram's crafting API (`/api/v1/crafting` → 8 skills: smithing, chiseling, fletching,
  crafting, alchemy, cooking, milling, smelting) has **no station/tool field** — recipes
  only expose `category`, `level`, `experience`, `requirements[]`, `result`, and sometimes
  `chance`/`tickRate`/`quest`. Asked the user how to handle "tool/station" given that gap;
  decided to show the crafting skill + level only, not a fabricated station name (e.g.
  "Smithing → Anvil" would be a guess, not sourced from the API).
- Minor known discrepancy: `/api/v1/crafting/count` reports 497 recipes but only 494 unique
  item keys end up in the merged `src/data/recipes.json` (a few items are apparently
  craftable via more than one skill, and the flat map keeps the last one). Same class of
  "API count vs. actual keys" quirk as the earlier item-count mismatch — not investigated
  further since it affects only a handful of edge-case items.
- Some recipe ingredients are `"#category"` tags (e.g. `#mushrooms`) meaning "any item from
  that group," not a specific item key. The API has no endpoint for these groups, so
  membership is derived in `scripts/fetch-kaetram-data.mjs` (→ `src/data/categories.json`)
  from whatever signal was actually verifiable — never guessed:
  - `#mushrooms`, `#mana_fruits` → happen to match drop-table names in `/api/v1/tables`
  - `#logs` → item keys starting with `logs`
  - `#vials` → exactly `vial_small`/`vial_large`
  - `#sand_or_glass_shards` → exactly `sand`/`glassshards`
  - `#stringables` → item keys starting with `bowunstrung`
  - `#large_potions`/`#small_potions` → per the user's explicit rule: finished (non-
    "unfinished") potions whose own recipe requires `vial_large`/`vial_small` directly
  - These render on item pages as "Any {label}" with an `<img>` that cycles through every
    member item's icon every 1.2s (`data-rotate-icons`, rotation script lives in
    `Layout.astro` so it works wherever `CategoryIngredient.astro` is used — item pages and
    the `/crafting` browser table).
- New `/crafting` page (linked from the header nav): tabs per crafting skill, each showing
  that skill's recipes as a table sorted by level requirement ascending. Reuses
  `CategoryIngredient.astro`/`ItemIcon.astro` for the ingredients column.
- The API's 8 crafting categories don't map 1:1 to real in-game skills — the user corrected
  this: Smithing = smithing+smelting, Crafting = chiseling+crafting, Fletching =
  fletching+milling (Alchemy and Cooking stand alone). `realCraftingSkillName()` in
  `src/lib/kaetram.ts` does this mapping; `/crafting` now shows 5 merged tabs (counts:
  Smithing 174, Crafting 128, Fletching 71, Alchemy 88, Cooking 33) with a small "Type"
  badge per row showing the specific sub-category (e.g. a Smelting item inside the Smithing
  tab). Item detail pages show "Skill: Smithing (Smelting)" when the two differ, or just
  "Skill: Smithing" when they're the same.

## Exchange (added 2026-07-20)
- New `/exchange` page — the **only** part of the site that isn't fully static-at-build-time.
  Kaetram's exchange API only supports per-item lookups (`/api/v1/exchange/item/{key}`), not
  a bulk "all current offers" listing, so it can't be baked in like everything else and stay
  fresh — market data changes far more often than the weekly rebuild. Confirmed
  `api.kaetram.com` sends permissive CORS headers (reflects any Origin), so the page fetches
  offers directly from the browser at view time rather than needing a Netlify Function proxy.
- Item search/filter on that page runs against a build-time item index (name + weaponType +
  icon path, ~1,834 items, embedded via Astro's `define:vars` — that forces the script
  inline/unbundled, so it's plain JS, no TypeScript, no npm imports inside it).
- "Item category" filtering: items have no category field in the API. Asked the user; went
  with the verified `weaponType` field only (sword/axe/bow/staff/spear/blunt/scythe/pickaxe/
  bigsword/shield, ~217 items) rather than guessing broader Weapon/Armor/Consumable buckets
  from the unlabeled `equipmentType` numeric code.
- Exchange availability status (`/api/v1/exchange/stats`) shown as a colored dot + label at
  the top of the page (green/available, red/unavailable, gray/unknown on fetch failure).
- Buy and sell offers render as two separate tables, each independently sortable by price or
  quantity via its own `<select>` (no page reload — offers are cached client-side after the
  initial fetch, sort just re-renders).
- **Price history chart** (added 2026-07-20): hand-rolled inline SVG line chart (no charting
  library — the page's script runs via Astro's `define:vars`, which forces it inline/
  unbundled, so no npm imports are possible there anyway) fed by
  `/api/v1/exchange/history/{key}?hours=N`. Time-range presets: 24h/7d/30d/90d/1y (default
  30d). Followed the `dataviz` skill: single series so no legend, 2px line + 10%-opacity
  area wash in the site's amber accent, hairline gridlines, hover crosshair+tooltip
  (value bold/leads, date secondary), a "View as table" fallback for accessibility.
  Also computes a quantity-weighted average price over the selected period and compares it
  to the current lowest sell offer (falling back to the last traded price if there are no
  live sell offers) to show a "N% below/above average" verdict — the "is this worth it"
  signal the user asked for. History records come back most-recent-first from the API;
  sorted ascending client-side before charting.

## Market analysis on Exchange (added 2026-07-20)
Extended the per-item exchange view with more analysis, all computed from data already
being fetched (no new endpoints):
- **Market stats tiles**: best buy offer, best sell offer, spread (abs + % of best buy),
  listed depth (total buy qty vs sell qty) — computed from the live offers, shown as soon
  as they load.
- **Market depth chart**: hand-rolled SVG, cumulative quantity by price tier, buy side
  (emerald) growing left from the spread, sell side (rose) growing right — mirrors the
  Buy/Sell color convention already used elsewhere on the page. X-axis is rank-ordered by
  price tier, not linear-by-price-value, so it stays readable when a market has a couple of
  extreme outlier offers.
- **Period stat tiles** (high/low, % change open→close, trade count, volume in qty and
  gold) computed from the currently-loaded history window — re-renders whenever the
  24h/7d/30d/90d/1y range changes.
- **Moving averages** on the price chart: 5-trade and 20-trade rolling averages, only
  rendered once there's enough history to be meaningful (≥8 records for the short one, ≥25
  for the long one) — deliberately trade-count-based rather than calendar-day-based, since
  trade activity per item is often too sparse for "7-day average" to mean anything. Adding
  these took the chart to ≥2 series, so per the `dataviz` skill a legend became mandatory
  (line-key swatches, not boxes) — only shown when at least one MA actually renders.

## Deployment
- Private GitHub repo + Netlify site, connected for Git-based CD — see decisions log
  2026-07-20 for the full walkthrough (site renamed once to strip the user's name from the
  default `<name>-eduardo.netlify.app` subdomain before going live).
- **Resolved 2026-07-20**: plain `git push` to `main` was blocked from 2026-07-20 03:37 to
  ~21:59 with `Build blocked: Unrecognized Git contributor. This plan allows only verified
  account members to push to private repos.` — a Netlify account/team setting, not anything
  fixable via CLI. The user fixed it on Netlify's side; confirmed working by a real
  push-triggered deploy (not a manual build-hook trigger) reaching `ready` state. Normal
  `git push` → auto-deploy now works as expected; no more manual `curl -X POST` to the build
  hook needed after ordinary pushes. (The weekly-rebuild function still uses the hook
  directly, which is correct and unrelated to this fix.)
- Netlify site ID: `afee928f-70b4-43e6-9685-226f57f5c6df`.

## Fixed: invisible text in Brave/Opera (2026-07-20)
A friend viewing the live site in Brave and Opera saw only the header — all other text was
present in the DOM (visible on select-all) but rendered the same color as the background.
Root cause: neither browser's forced/repainted dark-mode engine was told the page already
handles its own dark theme, so it recolored text and backgrounds independently and some
landed on the same value. The site never declared `color-scheme`. Fixed by adding
`<meta name="color-scheme" content="dark">` and `color-scheme: dark` in `global.css`. Not
an ad-blocker issue despite first appearances — verified the deployed page's actual DOM/CSS
in a standard Chromium browser before diagnosing, which ruled out a real build/content bug.

## Mob, boss, and skill icons (added 2026-07-20)
- Kaetram's API has no mob-icon endpoint, but `testwiki.kaetram.com` (their test wiki)
  serves mob sprites directly by internal mob key: `/sprites/mobs/{key}.png`. Confirmed
  this works before building anything (no HTML scraping needed).
- **Real bug found and fixed**: `testwiki.kaetram.com` is a client-routed SPA that returns
  HTTP 200 with its HTML shell for *any* unmatched asset path, not a real 404. The first
  version of the fetch script only checked `res.ok`, so every boss icon request (which
  404s at the regular mob path) silently got written as an HTML file with a `.png`
  extension — no crash, no error, just corrupted files sitting in the repo. Root-caused by
  checking actual response `Content-Type` headers, not status codes. Fix: `fetchBinary()`
  in `scripts/fetch-kaetram-data.mjs` now validates `Content-Type` starts with `image/`
  before accepting any download — applies to items, mobs, and skills alike.
- Bosses live at a **different path**: `/sprites/bosses/{key}.png`, not `/sprites/mobs/`.
  17 of 22 bosses resolve there; the other 5 (Balonus, Goblin Queen, Corrupted Cupidoid,
  Corrupted Slime, Evil Pumpkin) genuinely have no sprite published — tried several
  plausible path variants, all hit the same SPA fallback.
- Skill icons added from `/skills/{name}.png` (21 of 24; chiseling/smelting/milling
  unpublished) — wired into the item Requirements panel and crafting recipe skill line via
  new `SkillIcon.astro`.
- All genuinely-missing icons (5 bosses, 4 more regular mobs — several flagged
  `hideCollection: true` in Kaetram's own data, 3 skills) fall back to the existing
  letter-badge placeholder.

## Market Overview merged into Exchange (added 2026-07-20)
- Built a market-wide view (top gainers/losers by 24h % change, most-traded by 24h gold
  volume) as `/market`, then merged it into `/exchange` as the default "browse" state per
  user request — shown whenever there's no active search/filter and no item selected;
  selecting an item swaps in the full per-item live view with a "← Back to market overview"
  link. Old `/market` links 301-redirect to `/exchange` (`netlify.toml`).
- Same "no bulk endpoint" constraint as the rest of Exchange: market-wide requires polling
  every item individually, so it can't be live-per-visitor. Solved with a background job:
  `netlify/functions/refresh-market-data.mts` (background function, up to 15 min) polls all
  ~1,834 items' current offers, keeps the ~1,169 actively-listed ones, pulls 24h history for
  those, and writes a snapshot to Netlify Blobs (`market-data` store). A scheduled function
  (`trigger-market-refresh.mts`, hourly) kicks off the background function over HTTP, since
  scheduled functions are capped at 30s and can't be background functions themselves — the
  two-function relay pattern. `market-summary.mts` (plain sync function) serves the cached
  snapshot to the frontend at `/api/market-summary`.

## Item categorization (added 2026-07-20)
- First pass: weapon-type-only filter on `/items` (the only per-item category field the API
  exposes directly), mirroring the Exchange page's weapon filter.
- Second pass, pursued further: found Kaetram's **public source repo**
  (github.com/Kaetram/Kaetram-Open — the closed live server is a different, more advanced
  fork, but this public one shares the same data lineage) and pulled the actual `Equipment`
  enum from `packages/common/network/modules.ts` (`develop` branch, not `master` — `master`
  has a different/stale 8-value enum that doesn't match live data at all). Cross-validated
  the 12-value `develop` enum (0=Helmet, 1=Pendant, 2=Arrows, 3=Chestplate, 4=Weapon,
  5=Shield, 6=Ring, 7=ArmorSkin, 8=WeaponSkin, 9=Legplates, 10=Cape, 11=Boots) against real
  item data by sampling names per code — every sample matched (including two non-obvious
  cases: code 5 "Shield" mixes actual shields with off-hand tools like a fishing rod, and
  code 10 "Cape" mixes capes with quivers — both consistent with a shared functional slot).
  `itemCategory()` in `src/lib/kaetram.ts` implements this.
- Codes 12–15 postdate this public snapshot and aren't in the enum. Per the user's explicit
  direction: 12 = "Wings", 13–15 = "Cosmetics" (inferred from item-name patterns only —
  documented in code comments as unsourced, unlike 0–11).
- Category priority when building `/items`' filter: `weaponType` if present (more specific
  than the generic "Weapon" slot — e.g. "Sword" instead of just "Weapon"), else the
  equipmentType slot name, else "Other". 23 categories + Other, all 1,834 items accounted
  for (verified counts sum exactly).

## Open questions / next steps
- v2 candidate: leaderboards (also live/dynamic like Exchange, same "no bulk endpoint"
  constraint that limits Exchange to per-item lookups rather than market-wide views).
- The git-push-CD block (see Deployment above) is still unresolved — worth fixing in
  Netlify's dashboard so pushes don't require a manual build-hook trigger every time.
