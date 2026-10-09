# ContextStore

Accumulated context from my own projects, written so an AI agent can **reuse** it rather than
rediscover it — so the same mistake costs an afternoon once, not three times.

Everything in here was extracted from real work: the `DECISIONS.md` / `RUNBOOK.md` / `ROADMAP.md` /
`TASKS.md` files and commit histories of
[KubePlayground](https://github.com/Jovan253/KubePlayground),
[Music-Tool](https://github.com/Jovan253/Music-Tool),
[RobloxAppDigSell](https://github.com/Jovan253/RobloxAppDigSell),
RobloxClueless (private),
[Mythos](https://github.com/Jovan253/Mythos),
[portfolio](https://github.com/Jovan253/portfolio) and
[sudoku-solver](https://github.com/Jovan253/sudoku-solver).

**[→ Start at INDEX.md](INDEX.md)**

---

## Wire this into a project

Agents read the store **over the network** — there's no clone to keep in sync. Paste this into any
project's `CLAUDE.md` (or `AGENTS.md`):

```markdown
## Context Store

Before debugging something opaque, choosing a platform, setting up project scaffolding, or working
in an unfamiliar stack, consult my context store — it exists so the same mistake isn't made twice.

1. Fetch the index:
   https://raw.githubusercontent.com/Jovan253/ContextStore/main/INDEX.md
2. Match on **symptom** first, then stack. Fetch the one or two files that match
   (same base URL + the path from the index).
3. Tell me what you found and what you're applying.

Also worth fetching at the start of substantial work:
- `profile/working-agreement.md` — autonomy, git conventions, code style
- `profile/dev-machine-windows.md` — before writing shell commands or bumping a version

This repo's own instructions take precedence over the store. The store records what was true when it
was written — verify anything version-specific before relying on it.

After solving something non-obvious, add the lesson to the store (see its CLAUDE.md for the rules).
```

Prefer it applied everywhere automatically? Put the same block in `~/.claude/CLAUDE.md` instead and
every project inherits it.

## What's in here

| | |
|---|---|
| **[INDEX.md](INDEX.md)** | The routing table. Lookup by **symptom**, by stack, or by situation |
| **[profile/](profile/)** | Who I am, how I want an agent to work, what my machine is |
| **[stacks/](stacks/)** | Technology-keyed lessons — Kubernetes, Helm, AWS/Terraform, FastAPI, React/Vite, Postgres, free-tier hosting, Roblox/Luau |
| **[patterns/](patterns/)** | Cross-cutting: project documentation, a debugging playbook, the staged build/playtest loop, portfolio criteria |
| **[projects/](projects/)** | One page per repo: stack, current state, where its own docs live, what it taught |

The **symptom table in the index** is the single most useful thing here. Every row is a failure that
has already cost real time — an opaque OIDC rejection, a container crash-looping while the app is
fine, a React filter that works on exactly the first render, progress silently overwritten by a
retried save.

## Adding to it

Add an entry when you learn something that **would have saved time if it had been written down.**
Extend an existing file rather than creating a new one, lead with the symptom in the words it
actually appeared in, keep the measured numbers, and add a row to the index.

Full rules — including why no concrete infrastructure identifiers go in here — are in
[CLAUDE.md](CLAUDE.md).
