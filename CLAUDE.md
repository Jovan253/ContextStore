# CLAUDE.md — ContextStore

This repo is **Jovan's context store**: accumulated, hard-won context from his own projects, written
so an agent can reuse it instead of rediscovering it.

Two audiences, two jobs:

1. **An agent working in some *other* repo** → read `INDEX.md`, fetch what's relevant, apply it.
2. **An agent working in *this* repo** → add to the store without degrading it. Rules below.

---

## Consult the store before solving a non-trivial problem

**This applies to every project, not just this one.** Before you start debugging something opaque,
choosing a platform, setting up a new project, or working in an unfamiliar stack:

1. **Fetch `INDEX.md`:**
   ```
   https://raw.githubusercontent.com/Jovan253/ContextStore/main/INDEX.md
   ```
2. **Match on symptom first, then stack.** The symptom table is the highest-value part — these are
   mistakes that have already cost real debugging time.
3. **Fetch the one or two files that match.** Don't fetch the whole store; the index exists so you
   don't have to.
4. **Say what you found and what you're applying.** If the store had the answer, that's worth
   stating — it tells Jovan the store is earning its keep.

Also worth fetching unprompted at the start of substantial work:

- `profile/working-agreement.md` — how much to decide alone, git conventions, code style. **There is
  one repo where the autonomy rule is deliberately inverted.**
- `profile/dev-machine-windows.md` — before writing shell commands or recommending a version bump.

**Precedence:** the current repo's own `CLAUDE.md` / `AGENTS.md` beats this store. The store is prior
experience, not policy. And the store records what was true **when it was written** — if an entry
names a file, flag, or version, verify it still exists before relying on it.

## Adding to the store

Add an entry when you've just learned something that **would have saved time if it had been written
down** — a non-obvious failure mode, a platform constraint that forced a design, a measured number,
a correction from Jovan.

**Do not add:** anything already obvious from reading the code, general documentation a model already
knows, or facts that only matter inside one conversation.

### How

1. **Find the existing file it belongs in and extend it.** A new file is the last resort — a
   fragmented store doesn't get read. One file per stack, not per incident.
2. **Lead with the symptom, in the words it actually appeared in.** Future-you searches by symptom,
   not by cause. *"Uploads silently do nothing"* is findable; *"stale process on port 8000"* is not,
   because you don't know that yet when you're looking.
3. **State the cause, then the fix, then why it's that way** if the reason is interesting. The *why*
   is what makes it transfer to a different stack.
4. **Keep measured numbers.** "Throttled in 11 of 147 windows", "~25s on a T4", "~60 clicks to the
   first upgrade". Specific numbers are the most reusable thing in here.
5. **Add a row to `INDEX.md`'s symptom table** if it's a diagnosable failure. An entry nothing routes
   to is an entry nobody reads.
6. **Cross-link** with relative paths (`stacks/kubernetes.md`), and date anything environmental.

### House rules

- **No secrets, no credentials, no service-role keys.** Ever.
- **No concrete infrastructure identifiers** — this repo is public. Project refs, deploy URLs, load
  balancer hostnames, product IDs, job ids and account numbers stay out. Write the transferable
  lesson and point at the project's own repo for the specifics. (This is a deliberate decision, not
  an oversight.)
- **Write the lesson, not the narrative.** "X happens because Y" beats "I spent an afternoon on X".
- **Prefer deleting a stale entry over leaving it to mislead.** A wrong entry is worse than a
  missing one.
- **Convert relative dates to absolute.** "Last month" rots; "2026-09-26" doesn't.

## Layout

```
INDEX.md      ← the routing table. Every lookup starts here
profile/      who Jovan is, how he works, what his machine is
stacks/       technology-keyed lessons — the reusable bulk
patterns/     cross-cutting approaches that worked
projects/     one page per repo: stack, state, its own docs, what it taught
```

## Working in this repo

Normal autonomy applies (see `profile/working-agreement.md`): decide, apply, commit. Conventional
commit prefixes — `docs:` for content, `feat:` for structure.

**Keep `INDEX.md` in sync.** It's the entry point; if it drifts, the store stops working regardless
of how good the content is.
