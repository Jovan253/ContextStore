# portfolio

**Repo:** https://github.com/Jovan253/portfolio · **State as of:** 2026-09-27 (last push)

Jovan's personal portfolio site. **This is the shop window the other projects are stocking** — when
assessing whether a project is worth building or polishing, remember it ends up linked here.

There is also an older **`Portfolio-React`** repo (SCSS, last touched 2024); `portfolio` is the
current one.

---

## Stack

- React + JavaScript, Create React App lineage (`react-scripts`)
- `react-router-dom` for routing — the commit history shows a deliberate move to it
- Pages: hero/main, projects with hover states, an about section, a certifications page

## What the commit history tells you

It reads as a long series of small design corrections, which is itself informative about how he
works:

- Multiple rounds of formatting, colour (*"lighter blue"*) and comment revisions
- `fix: re-work sidebar`, `fix: new header` — layout iterated more than once
- `feat: mobile fixes and about section` — mobile was a deliberate pass
- `fix: add noopener` then `fix: noreferrer fix` — **external links were given
  `rel="noopener noreferrer"`**; keep that when adding links
- `fix: try to fix font for deployment` — a font that worked locally and not deployed
- **Projects are added here as they're finished:** Mythos (Aug 2026), KubePlayground (Sep 2026).
  Certifications too — AI-103 in Sep 2026.

## When working here

- **Finishing a project includes adding it here.** It's part of the definition of done, based on the
  pattern.
- Descriptions get revised for clarity — `feat: new project and better descriptions`. Expect to
  iterate on copy, not just ship it.
- No `CLAUDE.md`, no task file. See `patterns/project-documentation.md` if establishing one.

## Related

- `patterns/portfolio-project-criteria.md` — why these projects are shaped the way they are, and
  why a public demo matters more than a feature
