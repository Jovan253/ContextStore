# Working agreement

How Jovan wants an agent to behave. Drawn from the `CLAUDE.md` / `AGENTS.md` files in his own repos
and from the commit record.

**Scope note:** the autonomy and git permissions below were written in **Music-Tool's** `CLAUDE.md`.
They are a strong signal of preference, but **a repo's own `CLAUDE.md` wins** — check for one before
assuming push rights.

---

## Autonomy — the headline

> *"Make implementation decisions without asking for confirmation. When you have a clear
> recommendation, apply it. **Do not present options and wait — just do the better thing.**"*

This is the most important line in this file. Presenting a menu when you already know the answer
reads as friction, not diligence. Surface the decision you made and why, in the work itself.

Ask when the answer genuinely isn't derivable — a product/design preference, something irreversible,
or a choice that only he has the context for.

## Git

> *"Commit and push without asking for confirmation. Stage specific files (not `git add -A`), write a
> concise commit message, and push to `origin/main`."*

- **Stage specific files.** Never `git add -A`.
- Conventional-commit prefixes, used consistently across his repos: `feat:`, `fix:`, `ci:`, `docs:`,
  `chore:`, `refactor:`, with an optional scope — `feat(web):`, `chore(openspec):`.
- In the Roblox repo the style is instead an **imperative sentence describing the outcome** —
  *"Stop the case reveal from spoiling itself early"* — with a **body explaining the root cause**.
  Match the repo you're in.
- **Commit bodies carry the diagnosis.** His best commits explain *why*, not *what* — and that is
  where a lot of this context store came from. Keep doing it.
- Confirm push permission per repo if there's no `CLAUDE.md` saying otherwise.

## Code style

> - *"No comments unless the reason is non-obvious"*
> - *"No docstrings"*
> - *"No trailing summary paragraphs at the end of responses — the diff speaks for itself"*

That third one is about **responses**, not code. Don't close with a recap of what you just did.

## File operations and shell

Read, edit and write files — including creating new ones — without asking. Pre-approved commands are
listed per repo (installs inside the project venv, `npm run *`, dev server start/stop,
`docker compose up/down`, and any read-only inspection like `git log` / `git diff` / `git status`).

## The exception: learning repos

**KubePlayground inverts the autonomy rule, deliberately.** His words at kickoff:

> *"I want you to help me with Kubernetes but not do all of it."*

So for that repo:

- **Explain the concept, name the file to create, describe what it must contain *conceptually*** —
  which fields, why each exists, what breaks without it. **Then stop and wait.**
- **He writes the Kubernetes manifests himself.** Review and correct afterwards.
- **Do not pre-emptively create `.yaml` files.**
- Application code (Python backend, frontend JS) *may* be written for him — that's plumbing, not the
  thing he's trying to learn.
- Reviewing, debugging his errors, and **quizzing him** are all in scope and welcome.
- If he asks outright for a manifest, give it — but flag what he should understand about it.

**The general rule behind it:** when a repo's purpose is for *him* to learn a specific skill, handing
over the finished artefact defeats the point. Ask which half is the learning target if it's unclear.

## Documentation discipline

- **Keep the live task file in sync with reality** — *"it should always reflect what's actually built,
  not what was originally planned."*
- **Distinguish built from verified.** Leave an unticked line for an unconfirmed playtest.
- **Record symptoms in the words they appeared in**, and the user's verdicts verbatim with dates.
- **Convert relative dates to absolute** when writing anything down.
- Plans go in `plans/<date>-<slug>.md`, versioned in the repo, and are **frozen once work starts**.
- Full layout in `patterns/project-documentation.md`.

## Things he has pushed back on

Worth knowing, because each is a correction he had to make:

- **Blind guessing.** Two wrong guesses at a value → *"stop guessing blind"*. Change the loop
  instead; put him in it if the signal is visual.
- **Presenting options instead of deciding.** See Autonomy.
- **Mixing conceptually separate things into one UI** because it was convenient to implement.
- **Trailing summary paragraphs.**

## Related

- `profile/about-jovan.md` — experience level, how to pitch explanations
- `patterns/debugging-playbook.md` · `patterns/project-documentation.md`
