# Roblox and Luau

From **Dig & Sell** (repo `RobloxAppDigSell`) — a Rojo-synced Luau simulator, built and published
live in Sep 2026. Nearly every item here was found by playtesting, not by reading docs.

Setup: Rojo (`default.project.json`, `aftman.toml`), Studio for level geometry, server
services + `*.client.luau` controllers.

---

## Engine behaviours that are not obvious

- **Default controls stop evaluating `ClickDetector` hover and clicks entirely once *any* `Tool` is
  equipped.** This is the big one. "Require the pickaxe equipped" and "mine via ClickDetector" could
  never have worked together — the tell was that the cursor stopped reacting to hover the moment a
  tool was equipped.

  **The architecture that works:** drop ClickDetector entirely. Fire on the tool's `Tool.Activated`
  → a `RemoteEvent` → the server picks the nearest valid target within N studs of the player.

- **`CanQuery` defaults to true, independent of `CanCollide`.** A held tool's own parts will block a
  click raycast from ever reaching what's behind them. Set `CanQuery = false` on tool parts. (Note:
  this was a *real* bug, but fixing it did **not** fix the mining — the ClickDetector issue above was
  the actual cause. Two real bugs stacked. Don't assume the first fix was the whole story.)

- **`ParticleEmitter` needs a `Texture` to render anything.** There is no usable engine default;
  betting on one gives you invisible particles and no error. Alternatives that need no asset:
  plain `Neon` Parts flying outward and fading, or the built-in **`Sparkles`** instance.

- **`ClearAllChildren()` on a container also destroys the `UIListLayout` parented under it** — layout
  objects are children too. Symptom: every item stacks on top of the others at (0,0) and nothing
  scrolls, on the *second* and subsequent builds. Destroy only the children you created
  (e.g. by name), and leave layout instances alone.

- **Anchored parts moved via `CFrame` carry a standing player automatically** — same as any Roblox
  moving platform, no extra work.

- **The default PlayerList overlay can hide custom UI buttons.** Disable it if a button "isn't
  appearing".

## Toolbox assets

- **A free asset is often a `Model`, not a `Tool`.** Setting `CanBeDropped` on it crashes, because
  that property doesn't exist on a Model. Build a real `Tool` *around* the Model's contents: pick a
  `Handle` part, weld the rest to it.
- **Strip what came bundled** — remove any scripts inside the asset, and disable `CanQuery`.
- Prefer adapting a free asset over building geometry procedurally; the procedural pickaxe looked
  like a torch, the Toolbox one looked "really cool".

## Client/server split and the reveal-animation trap

- **`PlayerData` replicates to the client the instant the server grants something** — which is well
  before a 7-second reveal animation finishes. An inventory list that auto-refreshes on an ownership
  change will **spoil the reveal**. Fix: a `spinning` flag suppresses auto-refresh while the
  animation runs, then refresh manually once the tween actually completes.
- **The server commits the roll; the client animation always lands on that result.** There is then
  nothing to fake by watching the reel spin, and no need to trust the client at all.
- **A spin should land short of the end of the strip** (e.g. index 42 of 50), leaving decorative
  items visible past the marker once it settles — it reads like a wheel that could keep going rather
  than a strip that ran out.
- **Client-side barriers are the right tool for per-player lock state.** A translucent Part created
  client-side blocks *that* player until *they* unlock the zone, with no per-player CollisionGroups.
  Real security stays server-side in the unlock check — the barrier is a visual affordance, not
  enforcement.
- **The reactive service pattern worked well:** a service watches a `PlayerData` field (plus
  `CharacterAdded`) and applies the consequence, rather than being called imperatively from
  wherever the purchase happened. Used for pickaxe tier, equipped skin, and walk speed. New
  cosmetic/stat systems slot in by copying it.

## DataStore — the data-loss bug worth never repeating

**A DataStore load failure, including a transient one, was silently treated as "new player" — and
then the autosave wrote that empty profile over real progress.**

The fix is two parts:
1. **Retry** the load.
2. **Kick the player instead of resetting** if it still fails. A kicked player is annoyed; a wiped
   player is gone.

Also: **write a migration path for every field you add to a profile**, because saves exist from
before the field did. Every stage of this project added fields and every one needed it.

## State scoping

**Checkpoint tracking was keyed only by player, not by player *and* zone.** Invisible with one
parkour zone. With two, falling in the second zone before touching any of its checkpoints sent the
player back to wherever they last checkpointed in the *first* one.

The general shape: **state about "where the player is in X" must be keyed by X**, and the bug is
invisible until a second X exists. Worth scanning for proactively when you add the second instance
of anything.

Related: **a static checkpoint tied to a now-moving platform drifts out of alignment.** Size it to
cover the platform's full swing range and anchor it at the swing's centre.

## Studio / Rojo workflow

- **Guard world-building against leftovers.** The service only builds its zones folder if that
  folder doesn't already exist in Workspace — so a leftover folder in Studio's **Edit-mode**
  Workspace makes the old map keep showing no matter what the code says. Delete it manually.
- **For visual/spatial values, stop guessing from code.** Two blind guesses at a grip rotation axis
  (Z, then X) were both wrong and X made it visibly worse. The fix was to **have the user live-tune
  it in the Command Bar and report the value**, then bake it in — one round trip instead of N. See
  `patterns/debugging-playbook.md`.
- Live-sync via Rojo is not the published place. The first publish is its own event; test the live
  version separately.

## Monetization

- **Scaffold products before the IDs exist.** Keep a config table that is empty until the Creator
  Dashboard IDs are supplied, and have the UI **only render entries that have a real ProductId**.
  Work isn't blocked on a dashboard visit.
- **Grant everything through the single `ProcessReceipt` path**, regardless of what's being bought.
- **Separate currencies deserve separate shop UIs.** Mixing a gem-priced item into the cash shop
  "felt out of place"; splitting them, colour-matched to each currency's HUD readout, fixed it.
  Currencies earned through different loops should look different.
- **Bulk tiers should be ~25% better value per unit** — that shape reads as a deal.
- **An admin config (`Config.AdminUserIds` + an `IsAdmin` check)** that unlocks everything on join is
  worth half an hour early. Make it a one-line toggle so you can also test *as a normal player*. Note
  it can't cover real Robux purchases.

## Game design judgements that held up

- **Scale difficulty knobs by what else they affect.** The top WalkSpeed tier was kept deliberately
  modest (~75% increase) because WalkSpeed also increases jump distance and would have trivialised
  tuned parkour content.
- **Hand-set numbers beat formula-derived ones** when you want specific values to land exactly
  (a hub ore worth exactly 1, a top-zone ore worth 15). A formula fights you.
- **Make the reward a claim, not an auto-grant.** Discovery stayed automatic; claiming the reward
  required opening the Journal and clicking. The player gets an action and a reason to open the UI.
- **Undiscovered entries show `???`** rather than being hidden — a collection becomes an incentive to
  explore rather than a checklist.
- **Refund duplicates** rather than granting nothing — a dupe that pays something isn't a dead pull.
- **Assess balance with real numbers from config**, not vibes: ~60 clicks to the first upgrade,
  ~500× expected-value range across the full progression. That's how you find the actual risk — here,
  that one-click-destroy mining with few ore slots meant multiplayer contention could stall players.
- **Destroy-and-relocate beats respawn-in-place.** Ore reappearing in the exact same spot felt dead;
  respawning at a random slot from a fixed pool made the world feel alive.
- **Add a timing element when a distance element is already tuned out.** Jump platforms went from
  "too hard" → eased → "too easy" → oscillating side-to-side with randomised period and phase.

## Related

- `patterns/staged-build-and-playtest.md` — the build/playtest rhythm this project ran on
- `projects/roblox-dig-and-sell.md`
