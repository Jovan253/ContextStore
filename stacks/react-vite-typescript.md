# React, Vite and TypeScript

From **TrackSplit** (`Music-Tool`, React + Vite + TS, Vercel), **Mythos** (React 19, Vite 8,
force-graph) and **portfolio** / **sudoku-solver** (Create React App era).

---

## StrictMode double-invoke — three separate bugs in one project

React 18+ StrictMode mounts, unmounts and remounts components in development. This produced three
distinct bugs in TrackSplit's audio mixer, each fixed separately:

- A `ready` handler fired for a **stale** instance after remount → needed an `active` flag so
  handlers from the discarded mount are dropped.
- A ref slot wasn't cleared when a WaveSurfer instance was destroyed → the ref pointed at a dead
  object.
- A `readyCount` counter incremented from async callbacks never reached the expected total, because
  the remount reset surrounding state but callbacks from both mounts still fired.

**The generalisable rule: don't count async events to decide readiness.** The fix that stuck was
replacing `readyCount` with `wsRefs.every(...)` — i.e. **derive the state from the things
themselves rather than tallying the events that created them.** Counters incremented by callbacks
are the specific shape that breaks under StrictMode.

Corollary: if a bug only appears in dev and vanishes in a production build, suspect StrictMode
before you suspect the library.

## Library objects that mutate your data

**`react-force-graph` rewrites `link.source` and `link.target` from id strings into node object
references** after the first render. So this works once and then silently stops filtering:

```ts
// breaks after the first render
links: graphData.links.filter(l => visibleIds.has(l.source) && visibleIds.has(l.target))

// correct — normalise the endpoint to an id either way
links: graphData.links.filter(l => visibleIds.has(linkEndpointId(l.source)) && visibleIds.has(linkEndpointId(l.target)))
```

Symptom in Mythos: graph edges stayed visible after their nodes were filtered out. Worth a general
suspicion — **force-directed and canvas libraries often mutate the data you hand them in place.**

## Vite environment variables

- **Every `VITE_`-prefixed variable is baked into the client bundle at build time.** Changing one in
  a hosting dashboard requires a **redeploy**, not just a save. This catches everyone once.
- Equally: a `VITE_` variable is **world-readable**. Never put a service-role key or any server
  secret there — only publishable/anon keys.
- **Vite reads `.env` at startup**, so restart the dev server after editing it.

## Node and Vite versions — check per project, don't assume

- **Vite 8+ requires Node 20.19+.** TrackSplit is pinned to **Vite 5.x** because its machine was on
  Node 20.11/20.18, and upgrading would break the build.
- But **Mythos runs Vite 8 with TypeScript 6 and React 19.** So the constraint is per-environment,
  not a universal rule — **read the project's own lockfile and the installed Node version before
  advising an upgrade.** Two of these repos disagree on purpose.

## Vercel

- **Set Root Directory in the dashboard for a monorepo.** It is the one setting `vercel.json`
  cannot supply, and getting it wrong is the usual cause of a failed first build.
- Pin everything else — framework, build command, output dir, SPA rewrite — in `vercel.json` so the
  dashboard only holds the root directory and env vars.
- **Preview deployments get their own URLs**, which a `CORS_ORIGINS` allowlist will not match.
  Either add them as needed or accept that only production talks to the API.
- After the production URL exists, **add it to the API's CORS allowlist** or every request fails in
  the browser with an opaque CORS error.

## Judgement calls worth reusing

- **No router for two routes.** A path check plus an SPA rewrite does it; the dependency isn't
  earned. (Mythos, with real navigation, does use `react-router-dom` — the point is to decide, not
  to default.)
- **Make a component's data source injectable** so an authenticated view and a public demo share one
  implementation instead of two that drift. This is what let a full backend migration land with
  **zero frontend changes**.
- **One dark theme, no light variant**, when the subject matter justifies it. A light version of an
  audio tool reads as a generic web form.
- Use **one colour per channel/entity across every representation of it** (the waveform *and* its
  marker), and dim things that are inactive *because of another thing* — so it's obvious **why**
  they're inactive, not just that they are.

## Related

- `stacks/free-tier-hosting.md` — the CORS/secret/redeploy loop in full
- `patterns/portfolio-project-criteria.md` — why the frontend carries disproportionate weight
