# Postgres, SQLAlchemy and Alembic

Short file, but every entry is a bug that actually shipped. From **TrackSplit** (`Music-Tool`).

---

## Silent semantic traps

- **A SQLAlchemy `JSON` column stores Python `None` as JSON `null`, not SQL `NULL`.** So
  `column.isnot(None)` **matches every row**, including the ones you think are empty. This made a
  retention sweep re-expire already-expired jobs forever. If you need a real SQL `NULL` in a JSON
  column, you have to be explicit about it — and if you need "is this empty", test the JSON value,
  not nullability.

- **`update(field=None)` meaning "leave unchanged" means the field can never be cleared.** A retried
  job showed `done` next to a stale error message, because there was no way to express "set error
  back to null". Use a sentinel, or separate "set" from "clear". Covered also in
  `stacks/python-fastapi.md`.

## Drivers and connection strings

- **SQLAlchemy 2.1 changed the default driver for `postgresql://` from psycopg2 to psycopg 3.** The
  app wouldn't start after four months dormant — nothing in the code had changed. Pin SQLAlchemy, or
  name the driver explicitly in the URL (`postgresql+psycopg2://`).
- **Normalise the URL you're given.** Providers hand out `postgres://`, `postgresql://`, and
  sometimes URLs with extra path suffixes. Normalising once at config load means any provider's
  connection string works without a per-provider branch.
- **Set `pool_pre_ping=True`** against any auto-suspending serverless Postgres. It is what makes the
  resume invisible to the application instead of surfacing as a dead connection on the first query
  after idle.

## Migrations

- **Never run migrations on application startup** if the app can scale to zero or run multiple
  replicas — several containers will start at once and race. Use an explicit, deliberately-invoked
  one-shot. It also makes the deploy sequence legible: migrate, then deploy.
- Keep the current revision written down somewhere a human reads (a runbook status table), so
  "is the database at head?" is answerable without a shell.

## Destructive data jobs

- **Dry-run first, against real data, and have it report what it *would* do.** There are no backups
  on free tiers. Give the function a `dry_run=True` parameter from the start rather than adding one
  after the first scare.

## Related

- `stacks/free-tier-hosting.md` — Neon vs Supabase Postgres and why auto-resume won
- `stacks/python-fastapi.md`
