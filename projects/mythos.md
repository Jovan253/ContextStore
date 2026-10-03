# Mythos

**Repo:** https://github.com/Jovan253/Mythos · **State as of:** 2026-08-17 (last push)

A mythology explorer built around an **interactive force-directed knowledge graph** — nodes by
category, filterable, with routed detail views.

Small and young (three commits), but notable for being his most **modern frontend toolchain**.

---

## Stack

- **React 19** + **TypeScript 6** + **Vite 8**
- `react-force-graph-2d` for the graph, `react-router-dom` 7 for routing
- **Tailwind 4** via `@tailwindcss/vite`
- **oxlint** rather than ESLint
- `tsx` for a `scripts/validate-data.ts` data-validation script — worth noting: the dataset has a
  validation step, run via `npm run validate`

**This is the repo that proves version advice must be per-project.** Mythos runs Vite 8 / TS 6 /
React 19 while TrackSplit is pinned to Vite 5.x for Node-version reasons. Check the lockfile and the
installed Node before advising either way. See `stacks/react-vite-typescript.md`.

## Lessons it contributed

- **`react-force-graph` mutates `link.source`/`link.target`** from id strings into node object
  references after the first render. Filtering links by raw endpoint value therefore works once and
  then silently stops — the fix was a `linkEndpointId()` helper used on both endpoints. The symptom
  was *"graph edges remain showing"* after their nodes were filtered out.

  Generalises to: **canvas and force-directed libraries often mutate the data you hand them
  in place.** Recorded in `stacks/react-vite-typescript.md`.

## Notes

- There is a `.claude/` directory, but it only contains a scheduled-tasks lock file — **no tracked
  project instructions.** No `CLAUDE.md`, no `README` beyond a stub, no task file. If you work here,
  consider establishing the layout from `patterns/project-documentation.md`.
- Linked from his portfolio site as a featured project.
