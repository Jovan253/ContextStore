# Project documentation pattern

Jovan's repos converge on a consistent set of documents, and the convergence is deliberate — the
mature ones (`Music-Tool`, `KubePlayground`, `RobloxAppDigSell`) all have it. **Reuse this layout
when starting a project, and populate the matching file rather than inventing a new one.**

The organising principle: **each document answers a different question, and they cross-link instead
of repeating each other.**

---

## The files

| File | Answers | Audience |
|---|---|---|
| `README.md` | What does this do, and how does a request flow through it? | A visitor |
| `CLAUDE.md` / `AGENTS.md` | How should an agent work in this repo? Autonomy, environment, style | An agent |
| `ROADMAP.md` or `TASKS.md` | What is actually built, and what's next? | Both — **the live source of truth** |
| `DECISIONS.md` | Why is it built this way, what was rejected, what does it cost? | Future you, and an interviewer |
| `RUNBOOK.md` | Where does everything live and how do I run it from a cold machine? | Future you, months later |
| `plans/<date>-<slug>.md` | Frozen design rationale for one piece of work | Both |

### `CLAUDE.md` / `AGENTS.md`

Covers autonomy level, git permissions, which shell commands are pre-approved, code style, and the
environment's version pins **with the reason for each pin**. See `profile/working-agreement.md` for
the content that generalises across repos.

**`.claude/` is gitignored, so project-level Claude config does not travel with the repo.**
`CLAUDE.md` at the root is tracked, which is the part that matters — put anything you want to
survive a fresh clone there.

### `ROADMAP.md` / `TASKS.md` — the live one

State at the top that **this file is the source of truth for what's done and what's next — read it
before assuming where things stand.** Both KubePlayground and the Roblox project say a version of
this, and it's the single most useful line in the repo for an agent.

What makes these work in practice:

- **Milestones with a status legend** (`☐ not started · ◐ in progress · ☑ done`) — because "in
  progress" is most of the truth, and a binary checkbox hides it.
- **Lessons recorded inline against the item that taught them**, not in a separate wiki. The
  KubePlayground roadmap is as much a learning log as a plan, and that's why it's valuable.
- **Dated session entries, with the user's verbatim verdict** — *"looks great"*, *"slightly too
  hard"*, *"barrier is fine"*. Paraphrasing loses the signal.
- **Deferred ideas kept with the reasoning**, not deleted. *"Zones aren't big enough yet to need
  fast travel; revisit once zones grow."* That's a decision, not a dropped task.
- **Explicit ⚠️ blocks when the environment changes underneath old notes**, saying which earlier
  entries can no longer be trusted. Without this, a roadmap rots silently.

### `DECISIONS.md`

Each entry: **the choice, what was rejected, and what it costs.** The trade-off is usually the
interesting part, and writing the cost down is what stops you relitigating.

Three sections worth copying wholesale:

- **"Things that went wrong, and what they taught"** — a `| Symptom | Cause |` table. This is the
  highest-value artifact in any of these repos, and it's what most of this context store is built
  from. Write the symptom as it actually appeared, not as you understood it afterwards.
- **"Known limitations"** — *"being able to name these matters more than pretending they do not
  exist."*
- **"Questions worth having an answer ready for"** — the interview/review questions the architecture
  invites, answered. Forces you to notice the ones you can't answer.

### `RUNBOOK.md`

Explicitly *"written for coming back to this project after months away."* That framing is what makes
it useful, and the dormancy problem is real — this stack broke in three separate ways after four
months idle.

- **A "Status — read this first" table with a `Last verified` date.** Every row is a component and
  its actual state. An agent reads this first and skips a lot of discovery.
- **A "your values" table** — which identifier comes from which dashboard, filled in once. States
  plainly which values are safe to commit (project refs, anon keys) and which are not.
- **Service map:** what each service does, its dashboard URL, free-tier limits, and
  **"check when broken"** — the specific things to look at. Not generic advice.
- **Cold start from nothing**, as numbered commands, including system installs that aren't in any
  lockfile.
- **Troubleshooting keyed by symptom**, in the user's words. *"Uploads silently do nothing."*
- **A "gotchas worth remembering" section** for things that fit nowhere else.

### `plans/<date>-<slug>.md`

Versioned **in the repo**, not only in a tool's local plan-mode storage, so they persist across
machines and sessions.

**A plan is a frozen rationale document.** Once milestones are underway, track live status in
`TASKS.md` — **don't edit the plan.** The plan records what you believed at the time; that's its
value.

## Diagrams

**Mermaid, not images** — GitHub renders it inline and it stays diffable. Architecture diagram plus
a sequence diagram for the main flow. Note in the roadmap when a pending change will invalidate them.

## Two warnings

- **Checkmarks lie.** From TrackSplit's own runbook: *"Two OpenSpec changes are unarchived with all
  tasks ticked including production smoke tests that were never confirmed. Treat their checkmarks as
  unverified."* Distinguish **built** from **verified**, and say who verified it and when. The Roblox
  TASKS.md does this well — every playtest line records the date and the verdict.
- **A document nobody reads first is wasted.** The reason these work is that `CLAUDE.md` points at
  the live file, and the live file points at the frozen ones. Keep the entry point single and
  obvious.

## Related

- `profile/working-agreement.md` — what goes in the `CLAUDE.md`
- `patterns/debugging-playbook.md` — the symptom/cause table is this pattern's main output
