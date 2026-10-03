# sudoku-solver

**Repo:** https://github.com/Jovan253/sudoku-solver · **State as of:** 2026-03-07 (last push)

*"Solve Sudoku Visually"* — a React app that animates a backtracking solver so you can watch it work.

An early, small project. Included here mainly for completeness and because it's linked from the
portfolio.

---

## Stack

React + JavaScript, Create React App (`react-scripts`). No TypeScript, no test suite, no task file,
a one-line README.

## Notes

- Five commits total, including the CRA scaffold — effectively a weekend project.
- One commit is `fix: push to main` followed by `fix: revert change`, so there was a brief branch/
  push mix-up early on; nothing structural.
- **Visualising an algorithm's intermediate state is a recurring instinct of his** — the same impulse
  shows up in KubePlayground (the ownership tree, the kubectl transcript, the spec-vs-stored YAML
  pane) and Mythos (the force graph). Worth knowing when proposing how to present something: he
  reaches for *watch it happen* over *read the result*.

## If picking this up again

It's a reasonable candidate for a small modernisation (CRA is the dated part) or for being the
subject of the kind of documentation pattern in `patterns/project-documentation.md`. But check it's
actually worth the effort against `patterns/portfolio-project-criteria.md` first — a polished solver
is not a differentiator next to KubePlayground or TrackSplit.
