# Staged build and feedback loop

The working rhythm that produced **Dig & Sell** from nothing to published in roughly three days, and
then through seven follow-up rounds plus a two-stage Gem system. It generalises to any project where
the human is the one who can judge whether it's good.

---

## The loop

1. **Discuss until the design is actually finalised**, then log the finalised version with a date
   and a note of how many discussion rounds it took.
2. **Stage the build.** Core system first, the things that spend/consume it next round. Say so
   explicitly up front.
3. **Build one stage.**
4. **Hand it over for a live playtest.** Say what to look at.
5. **Record the verdict verbatim, with the date.** *"looks great"*, *"slightly too hard"*,
   *"barrier is fine"*, *"works a charm"*, *"looks ok for most part"*.
6. **Fix what came back in the same round**, then re-confirm.
7. Only then start the next stage.

The staging is the important part. *"Build core system first, spends next round"* was used for both
the Gem system and the zone/parkour work, and it kept each round small enough to playtest in one
sitting.

## Why the verbatim verdict matters

*"Slightly too hard"* → eased → *"too easy"* → **added a timing element instead of changing the
distance again.** That third move only exists because both earlier verdicts were recorded in the
user's own words. A checkbox marked "done" would have lost the whole arc.

Paraphrasing a verdict into "feedback addressed" destroys the information you need two rounds later.

## Build vs verified are different states

Every item in that project's `TASKS.md` distinguishes them, and the ones awaiting confirmation stay
visibly unconfirmed:

```
- [x] Gem Packs live: Config.GemProducts, MarketplaceService grants via ProcessReceipt
- [ ] Not yet playtested live
```

That second line is doing real work. Compare the warning in TrackSplit's runbook about tasks ticked
including *"production smoke tests that were never confirmed — treat their checkmarks as
unverified."*

## Record deferrals with their reasoning

Not deleted, not silently dropped — kept with *why*, and with whose idea it was:

- *"Fast travel: zones aren't big enough yet to need it; revisit once zones grow."*
- *"Sell Shop: good pattern, but changes the core loop's reward timing across a lot of
  already-tuned/tested code. Own future round."*
- *"Playtime trickle dropped — not reaffirmed in the final discussion, keeping this stage's scope
  tighter."*
- *"Rendering an actual pickaxe image instead of a colour swatch — explicitly a nice-to-have, not
  requested as a follow-up."*

Each of those is a decision with a trigger condition. A deleted line is just lost context.

## Separate what the agent can do from what the human must

Flag the human's actions explicitly and don't let them block the build:

- *"**User action needed**: set Max Player Count in the Dashboard, no code involved."*
- *"Icon/thumbnail need to come from the user — can't generate or screenshot them myself."*
- Product IDs had to be created in a dashboard, so the code shipped with an **empty config table and
  a UI that only renders entries with a real ID.** Work continued; wiring was a one-line change later.

That last pattern is the good one: **scaffold against the missing external thing so its absence
doesn't block you.**

## Assess with numbers, not vibes

When asked whether the pacing was right, the answer came from computing it out of the config: ~60
clicks to the first upgrade, ~500× expected-value range across the full progression. That's what
surfaced the real risk — **one-click-destroy mining with few ore slots meant multiplayer contention
could stall players waiting for respawns**, which no amount of eyeballing would have found.

**Derive the numbers from the config and state them.** It changes the conversation from opinion to
arithmetic.

## When the feedback loop is the bottleneck, change the loop

Two blind guesses at a rotation axis cost two full round trips and made it worse. Switching to
*"you live-tune it in the Command Bar and tell me the value"* solved it in one. See
`patterns/debugging-playbook.md` §3.

## Related

- `stacks/roblox-luau.md` — what this loop actually produced
- `patterns/project-documentation.md` — where the rounds get recorded
- `projects/roblox-dig-and-sell.md`
