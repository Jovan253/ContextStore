# Clueless: Lava Rising

**Repo:** `Roblox-Clueless` (local, no remote yet) · **State as of:** 2026-10-09

A Roblox take on the browser game **Clueless** (one secret word; every guess is ranked by how close its
*meaning* is, rank 1 = solved). Each player stands on a pillar in a volcano crater; the pillar's height
follows their best rank, and lava rises steadily — stop getting closer and it catches you. The first
solve rockets that player to safety and starts a 20 s finale for everyone else.

The first project built **Studio-first through the Roblox Studio MCP** (no Rojo) — see
`stacks/roblox-luau.md` → *Recommended setup*.

---

## Stack and structure

- Live Studio place is the source of truth; the repo holds docs, tools and a read-only
  `studio-mirror/` of every script (`tools/sync_mirror.py` refreshes it from a session transcript)
- Server services in `ServerScriptService.Services` (`WordService`, `PlatformService`, `LavaService`,
  `RoundService`) started from one `Main` script; shared `Config` + `Remotes` modules (same pattern as
  Dig & Sell)
- Round/player status replicates via attributes (`ReplicatedStorage.RoundInfo`, Player attributes);
  remotes only for guesses (private, back to the guesser), notices and the recap
- `tools/wordgen/build_words.py` — offline word-similarity pipeline (below)
- Blender + Blender MCP set up for the art; not used yet

## Its own docs

- `TASKS.md` — live checklist · `AGENTS.md` — Studio-first workflow rules
- `plans/2026-10-09-v1-roadmap.md` — frozen design doc (gameplay, monetization, milestones)

## What's built (2026-10-09)

Milestones 0–2: tooling, word pipeline, greybox crater/pillars/lobby, the full solo round loop and
HUD — all verified by MCP-driven playtests. Next: multiplayer + lobby spectating, Blender art,
DataStore economy, monetization (revive, hint, troll products, VIP, 2x coins).

## Lessons it contributed

- **Semantic similarity can't run inside Roblox — precompute it.** GloVe 300d
  (`glove-wiki-gigaword-300` via gensim, ~380 MB) → 20k-word vocabulary filtered by WordNet `morphy`
  + a profanity list → for each of 185 secrets, rank the whole vocab by cosine and keep the top 5,000 →
  one 8.4 MB `.rbxmx` inserted into ServerStorage. Guesses outside the top 5,000 are "cold". No server,
  no latency, no hosting cost.
- **GloVe quirks that matter for a game:** function words ("but", "so", "it") crowd the top of
  abstract secrets like *time*/*hand* — exclude stopwords from rankings; news-corpus senses leak in
  (*ring* → trafficking, smuggling; *star* → celebrity before galaxy) — drop or curate those secrets.
  The WordNet filter also means "the" isn't a valid guess at all.
- **Log-scale height:** `1 − ln(rank)/ln(5001)` so far-off early guesses still visibly move you. With
  lava at 0.25 studs/s accelerating 0.0015/s², you need roughly rank ≤350 by 2:00, ≤30 by 3:00 and a
  solve by ~4:00 (to be tuned with real players).
- The `WorldPivot`/`PrimaryPart` teleport bug and the MCP separate-VM gotcha — both in
  `stacks/roblox-luau.md`.
