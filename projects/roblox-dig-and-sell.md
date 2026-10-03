# Dig & Sell

**Repo:** https://github.com/Jovan253/RobloxAppDigSell · **State as of:** 2026-09-23 (last push)

A Roblox "dig & sell" miner simulator — mine ore, earn cash, upgrade your pickaxe, unlock zones,
rebirth. **Published live to Roblox on 2026-09-23.**

The fastest-moving project in the set: scaffolding to published game in about three days, then seven
follow-up rounds and a two-stage Gem system on top.

---

## Stack and structure

- **Rojo-synced Luau** — `default.project.json`, `aftman.toml`, source in `src/`
- Server services (`MiningService`, `DataService`, `ShopService`, `CollectionService`,
  `CaseService`, `SpeedBoostService`, `PickaxeToolService`, `LeaderboardService`,
  `EnvironmentService`) plus `*.client.luau` UI controllers
- `Config.luau` holds all tuning data — zones, ore tables with rarity weights, tiers, prices, skins
- `ZoneGeometry` — a shared orientation-aware local-to-world transform, so every service agrees on
  where a zone's entrance and footprint actually are
- **Map/level geometry is built visually in Studio, not in code**
- Persistence via DataStore + OrderedDataStore for the leaderboard

## Its own docs

- **`TASKS.md`** — the live checklist, and a genuinely excellent one. Every round is dated, every
  playtest records the verdict in Jovan's own words, every bug records its root cause.
- `AGENTS.md` — planning-doc conventions, where live status lives
- `plans/2026-09-21-mvp-roadmap.md` — the frozen design doc

## What's built

All ten MVP milestones, plus:

- **5 zones** in a radial hub-and-spoke layout, each sized to exactly match a hub side
- Two parkour zones — **Dark Mines** (lava) and **Sky Ruins** (bottomless drop) — with per-zone
  checkpoints and side-to-side oscillating platforms
- **Pickaxe as an equippable Tool** with swing animation and tier-scaled multi-hit mining
- **Monetization:** a 2x Cash gamepass, Cash Pack and Gem Pack developer products, a Jumpscare Prank
  product. All via one `ProcessReceipt` path.
- **Gem system v1, fully confirmed working:** 15 ore types across 5 zones × 3 rarity tiers, a
  **Collection Journal** with claim-based rewards, **Speed Boost** tiers, and **pickaxe skin cases**
  with a spinning reveal reel
- Leaderboard, rebirth, admin account toggle

**Open / deferred, with reasoning recorded:** Slap, Hover charges, Lucky Charm, Pets, Fast Travel
(zones aren't big enough yet), a Sell Shop (would change the already-tuned core loop's reward
timing), and rendering real pickaxe images instead of colour swatches in the case reel.

**Still on the user:** game icon and thumbnails (need real screenshots), and capping Max Player
Count to ~20–30 in the Dashboard.

## Gotcha if re-testing

`MiningService` only builds its zones folder if it doesn't already exist in Workspace. **If Studio's
Edit-mode Workspace has a leftover one, delete it manually** or the old map keeps showing no matter
what the code says.

## Lessons it contributed

- `stacks/roblox-luau.md` — nearly all of it: the ClickDetector/Tool conflict, `CanQuery`,
  `ParticleEmitter` textures, `ClearAllChildren` eating the layout, the DataStore data-loss bug,
  per-zone state scoping, and the game-design judgements
- `patterns/staged-build-and-playtest.md` — the whole working rhythm is derived from this project
- `patterns/debugging-playbook.md` — "two wrong guesses means change the loop", and "fixing a real
  bug doesn't mean you fixed *the* bug"
