# TrackSplit

**Repo:** https://github.com/Jovan253/Music-Tool · **State as of:** 2026-09-27 (last push)

Named **TrackSplit** in the repo; the GitHub repo is still `Music-Tool`. Splits an uploaded track
into four playable stems (Demucs) and gives you a browser mixer to play them against each other —
built for practising along to music.

**The most documented and most architecturally interesting of his projects.** If you need an example
of how he likes a project set up, this is it.

---

## The constraint that shaped everything

Two requirements pulling opposite ways:

1. **Stem separation needs a GPU** — ~25s on a T4, ~12 minutes on a laptop CPU
2. **It had to cost nothing** — *"a project that quietly bills £20/month while nobody visits gets
   deleted"*

Almost every decision follows from wanting **GPU compute on demand while paying nothing when idle.**

## Stack

Monorepo: `apps/api` (Python/FastAPI) + `apps/web` (React/TypeScript/Vite).

| Concern | Platform |
|---|---|
| API host + GPU separation | **Modal** — one platform, ASGI app + T4 function, scale to zero |
| Job queue | **None** — `.spawn()` replaced Redis + RQ entirely |
| Postgres | **Neon** — chosen over Supabase's because it auto-*resumes* |
| Auth | **Supabase** |
| Object storage | **Supabase** today; **Cloudflare R2** is the migration target |
| Frontend | **Vercel** |
| Previously | Railway — retired 2026-09-26 |

Stems stored as **256kbps MP3**, not WAV: ~10× faster waveform loads and ~20–60MB/job, which is what
makes a 1GB tier viable at all.

## Its own docs — unusually good, read them

- **`RUNBOOK.md`** — the best file in any of his repos. Status table with a `Last verified` date,
  service map with free-tier limits and "check when broken", credential provenance, cold-start
  instructions, troubleshooting by symptom.
- **`DECISIONS.md`** — choice / rejected / cost for every decision, plus a *"things that went wrong
  and what they taught"* symptom table, known limitations, and interview questions answered.
- `CLAUDE.md` — autonomy, Windows process gotchas, version pins with reasons
- `TASKS.md`, `README.md` (with Mermaid architecture + sequence diagrams), `openspec/`

## Where it got to

- **Live end to end, verified 2026-09-26:** authenticated upload → Modal `.spawn()` → T4 separation
  in ~7s → four playable stems via signed URLs
- **Public demo page** at `/demo`, unauthenticated, serving one pre-separated track — the
  highest-leverage portfolio item, deliberately built
- **Retention live** — a daily Modal cron expires jobs past a window
- 48 API tests, GitHub Actions running tests + web lint + build, green

**Open:** storage migration to R2 (credentials in place, no code reads them); no job history (the id
lives only in React state — putting it in the URL is the next fix); no observability; mobile layout
written but unverified; Railway project still needs deleting in its dashboard; the app's name isn't
settled.

## Lessons it contributed

The majority of this store's application-layer content:

- `stacks/free-tier-hosting.md` — platform comparison, scale-to-zero pitfalls, retention, secrets
- `stacks/python-fastapi.md` — Windows uvicorn, packaging extras, ML dependency traps
- `stacks/postgres-and-data.md` — the SQLAlchemy `JSON`/`NULL` trap, the 2.1 driver change
- `stacks/react-vite-typescript.md` — StrictMode races, Vite env baking, Vercel monorepo setup
- `patterns/project-documentation.md` — the RUNBOOK/DECISIONS pattern comes from here
- `patterns/portfolio-project-criteria.md` — *"nobody evaluating this will sign up"*
