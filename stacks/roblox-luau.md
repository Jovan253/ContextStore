# Roblox and Luau

From **Dig & Sell** (repo `RobloxAppDigSell`) — a Rojo-synced Luau simulator, built and published
live in Sep 2026 — and **Clueless: Lava Rising** (repo `Roblox-Clueless`), built Studio-first through
the Roblox Studio MCP from Oct 2026. Nearly every item here was found by playtesting, not by reading
docs.

**For a new Roblox project, use the Studio-first MCP setup below** — Jovan's call after both projects
(2026-10-09): *"it is proven to be more efficient"*. Dig & Sell's Rojo + code-built-geometry setup is
still described further down because that repo still uses it.

---

## Recommended setup: Studio-first through the Roblox Studio MCP

**Why:** Dig & Sell started on Rojo with geometry rebuilt by scripts on every server start. That
fought live Studio design work (a procedural rebuild wipes whatever was placed in the editor, and a
leftover Edit-mode folder silently kept the old map) and the project had to migrate to a static map
midway. With the MCP, Claude edits the **live place** directly: scripts, geometry, UI, playtests.

**Setup (Windows, ~10 min — split into what Claude runs vs. what Jovan clicks):**

1. Studio ships the MCP server itself: `%LOCALAPPDATA%\Roblox\mcp.bat` → `StudioMCP.exe`. Register it
   per project (Claude): `claude mcp add Roblox_Studio -- cmd.exe /c "cd /d %LOCALAPPDATA%\Roblox && .\mcp.bat"`.
   From Git Bash this needs `MSYS_NO_PATHCONV=1` — see `profile/dev-machine-windows.md`.
2. Jovan: open/create the place in Studio and make sure Studio's MCP server is switched on; restart
   Claude Code; `/mcp` shows `Roblox_Studio` connected.
3. Every tool call takes a `studio_id` from `list_roblox_studios` — several Studios can be open at
   once, so pick by place name.

**Workflow that worked:**

- **The place file is the only live copy of the code.** The repo holds docs, offline tools, and a
  **read-only `studio-mirror/`** of every script's `Source`, refreshed at each milestone: an
  `execute_luau` snippet returns all sources as one JSON blob, and a small script pulls the newest blob
  out of the Claude Code session transcript (`.jsonl`) and rewrites the mirror — no re-typing code
  through the model. Implementation: `tools/sync_mirror.py` in the Clueless repo.
- **Remind Jovan to save/publish every session.** Nothing the MCP does is on disk until he does.
- `multi_edit` with a `className` creates a new script **and any missing parent folders**
  (`ServerScriptService.Services.X` made the `Services` folder).
- Wrap Edit-mode world building in `ChangeHistoryService:TryBeginRecording` / `FinishRecording` so a
  whole build is one undo step for him.
- **Large generated data ships as an `.rbxmx` file** — plain XML, trivial to emit from Python
  (`Folder` → `ModuleScript` items with a `ProtectedString name="Source"`). Jovan right-clicks
  ServerStorage → *Insert from File*. 8.4 MB (a 20k-word list plus 185 modules of 5,000 lines each)
  inserted and `require`d without trouble. Put anything clients must not see in **ServerStorage**.
- **Drive playtests entirely through the MCP:** `start_stop_play`, then fire `RemoteEvent`s from the
  **Client** datamodel (`require(Remotes)` + `FireServer`) to simulate input, and poll state from the
  **Server** datamodel. A full round loop (guess → rise → solve → recap; idle → eliminated → respawn)
  was verified this way without touching the keyboard.
- `screen_capture` with `camera_position`/`look_at_position` frames shots in Edit mode; with no camera
  args during play it captures the player's own view — the cheap way to check HUD layout.

### MCP gotchas

- **"Module state reads `nil` when inspected through the MCP, but the game is clearly using it."**
  `execute_luau` runs in its own Lua VM, so `require(module)` returns a **fresh instance**, not the
  running game's. Inspect live state through replicated attributes / instances instead, or derive it.
  (The round's secret word was recovered by matching three observed guess ranks against the word data.)
- **Multiplayer tests work through the MCP — once Jovan clicks *Test → Clients and Servers*.** No MCP
  tool can start that test (`start_stop_play` is solo only), but every window it opens appears in
  `list_roblox_studios` as its own `studio_id` with `name: null`; `get_studio_state` tells them apart
  (*"Focused DataModel in the viewport: Server"* vs `Client`), and `Players.LocalPlayer.Name`
  (`Player1`/`Player2`) identifies each client. One agent then drives every window in sequence —
  guesses fired per client, server polled for state, screenshots per player. No need for one subagent
  per player; they'd share the same MCP connection anyway.
- **Testing "can players escape this area?" through the MCP:** `user_keyboard_input` W moves relative
  to the *camera*, so a held W + Space test was inconclusive. Reliable version, from the **Client**
  datamodel: loop `humanoid:Move(direction)` + `humanoid.Jump = true` for a few seconds and record the
  max position reached. (`character_navigation` answering *"Can not find a route to the destination"*
  is a quick first signal, but it's pathfinding, not physics.) And remember the obvious one it
  caught: **a 5-stud railing doesn't contain anyone — a default jump clears ~7**, and players can stand
  on the rail and jump again from there (peak root Y +17 above the floor). Use tall invisible walls
  with `CanQuery = false` so the camera and clicks ignore them, plus a server safety net that returns
  anyone who still falls.
- **`user_mouse_input` with an `instance_path` performs a real click on a GUI button** — it verified a
  Spectate button end-to-end, not just its handler.
- **The MCP undoes camera changes made in `execute_luau`** (*"The execute_luau changed camera type.
  Resetting from Enum.CameraType.Scriptable back to Enum.CameraType.Custom"*), so you can't script a
  camera angle for a screenshot that way. Screenshots of a play session also come back **black when the
  Studio window isn't focused/visible**.
- **Results over ~50 KB aren't returned inline** — Claude Code saves them to a `tool-results/*.txt` file
  and returns the path. Handy for bulk exports: hand the file to a script instead of reading it.
- **`execute_luau` freezes Studio until the code yields.** `task.wait()` inside is fine — 50 s polling
  loops watching a round worked — but bulk instance creation must be chunked with yields.

## Things to be wary of (Studio-first)

- **No git safety net for the place.** Revert = Studio undo or Roblox version history. Mirror often;
  commit the mirror; save/publish.
- **Never edit `studio-mirror/` expecting it to sync back** — it is an export, not a source.
- **Keep geometry static, scripts behaviour-only.** Scripts move pre-placed parts (pillars, lava); they
  never rebuild the map.
- **Hazards are server-authoritative.** Compare positions every `Heartbeat` (lava Y vs. platform top /
  root Y) instead of `Touched` — deterministic and testable: the idle player was eliminated at
  **36.0 s**, exactly when the lava crossed the platform's starting height.
- **The repo itself can leak answers.** Clueless's `secrets.txt` and ranking preview are every
  answer in the game; ServerStorage protects them from clients, a public GitHub repo would not.
  Keep game repos with answer/loot-table data private.
- **Size a "last N seconds" finale by the lowest player, not the highest.** Lava sped up so the *top*
  surviving pillar would be reached at 0:20; a player at mid height lasted **3.7 s** of a banner that
  said *"Solve it or burn 0:20"*. If the UI promises everyone a window, the hazard has to honour it.
  Fix that held up: a two-leg schedule — ease the hazard up to the **lowest** active player over a
  fixed grace (10 s), then sweep to the top by the deadline. Retested: the lowest player went at
  exactly **10.0 s**, and anyone who improves mid-finale buys themselves more time.
- **Anything a client must not know lives in ServerStorage**, and per-player secrets (e.g. a player's
  own guesses) go back only to that player via `FireClient`; everything shared rides replicated
  attributes, which need no remote plumbing.

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

- **"Everything is black except Neon"** after a lighting change: `Lighting.ClockTime` past ~18 puts the
  sun below the horizon, so only ambient lights the scene — and deleting the default `Sky` makes it
  worse. Dusk mood: ClockTime ~16–17 with a warm `Atmosphere` and `ColorCorrection`, not a later clock.
- **Lights that follow a moving part:** parent `PointLight`s to `Attachment`s on that part (the rising
  lava carries 11 of them). Mind the part's rotation — a cylinder lying on its side has local +X as
  world up.

- **`ParticleEmitter` needs a `Texture` to render anything.** There is no usable engine default;
  betting on one gives you invisible particles and no error. Alternatives that need no asset:
  plain `Neon` Parts flying outward and fading, or the built-in **`Sparkles`** instance.

- **`ClearAllChildren()` on a container also destroys the `UIListLayout` parented under it** — layout
  objects are children too. Symptom: every item stacks on top of the others at (0,0) and nothing
  scrolls, on the *second* and subsequent builds. Destroy only the children you created
  (e.g. by name), and leave layout instances alone.

- **Anchored parts moved via `CFrame` carry a standing player automatically** — same as any Roblox
  moving platform, no extra work. Confirmed again with a `PivotTo` every `Heartbeat` at up to
  **40 studs/s** upward: the character rode it, staying ~2.5 studs above the cap.

- **"A teleported character slides sideways off its platform within ~0.3 s and falls"** — or more
  generally, **a model lands in the wrong place after `PivotTo`**. Setting `Model.WorldPivot` does
  not stick once the model has a `PrimaryPart`: the pivot becomes `PrimaryPart.CFrame *
  PrimaryPart.PivotOffset`. A pillar's pivot stayed at the centre of its 300-stud column, so
  `PivotTo(y = 10)` put the cap 150 studs up — and the character teleported to "pivot + 4" landed
  *inside* the column, where physics depenetration shoved it out sideways. **Fix:** set `PrimaryPart`,
  then set `PrimaryPart.PivotOffset` (e.g. to the cap's top face), and assert `GetPivot()` right after.

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

## Blender → Studio (Blender MCP)

Set up 2026-10-09 for Clueless; **the art pipeline itself is not yet proven** — update this when it is.

- **The package was renamed `blender-mcp` → `mcp-for-blender`.** The old PyPI name still resolves but
  is stale (2.0.0 vs. 2.1.9 in the repo). Register: `claude mcp add blender -- cmd.exe /c uvx mcp-for-blender`.
- **`uvx mcp-for-blender install-addon` says "No Blender addons directories found"** on a fresh
  install: Blender's user config doesn't exist until Blender has run once. Create it headless:
  `blender -b --python-expr "import bpy; bpy.ops.wm.save_userpref()"` — then install, and enable
  headless too: `addon_utils.enable('blender_mcp', default_set=True, persistent=True)` +
  `save_userpref()`. The add-on starts its socket server (localhost:9876) when Blender opens, so
  Jovan only has to launch Blender.
- **On Windows the installer crashes with `UnicodeEncodeError: 'charmap' codec can't encode character '\u2192'`**
  after doing its work — set `PYTHONIOENCODING=utf-8`.
- Telemetry is opt-in and off by default; `get_addon_status` reports it.
- **Route into Roblox: FBX → Jovan clicks Studio *Import 3D*.** The Studio MCP can't import a local
  file (its asset tools are marketplace/generation only), so batch every model before asking — one
  interruption, not three. Model in Blender at **1 unit = 1 stud** and build geometry in Python
  (`bmesh`) through the Blender MCP; `look` renders a check without leaving the conversation.
- **"The imported mesh is enormous"** — the importer scales each model so its largest side fits the
  **2048-stud MeshPart limit** (a 300-stud pillar came in ×6.83, a 570-stud crater ×3.59, a 94-stud
  ledge ×21.8). Proportions survive, so record each object's Blender dimensions and set
  `MeshPart.Size` back to them.
- **"The model is facing the wrong way"** — exported with `axis_forward="-Z", axis_up="Y"`, Blender
  **+Y landed on Roblox +Z**, not −Z. A 180° turn about Y fixed it. Verify orientation with a downward
  raycast at a known landmark (rim height at the lobby notch) rather than by eye.
- **`MeshPart.CollisionFidelity` can't be set from a script** (not even plugin-level `execute_luau`) —
  only in the Properties panel. Default (convex decomposition) was fine for hex pillars; for a huge
  concave crater ring, keep it visual-only (`CanCollide = false`) and leave simple invisible Parts as
  the collision.

## Studio / Rojo workflow (Dig & Sell)

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
- `projects/roblox-dig-and-sell.md` · `projects/roblox-clueless.md`
- `profile/dev-machine-windows.md` — Git Bash path-mangling when registering MCP servers
