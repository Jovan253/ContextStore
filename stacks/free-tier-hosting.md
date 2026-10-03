# Free-tier and scale-to-zero hosting

From migrating **TrackSplit** (`Music-Tool`) off an always-on paid stack onto an all-free,
scale-to-zero one (Sep 2026). The constraint was explicit and worth restating because it drove
every decision:

> GPU compute on demand, while paying nothing when idle. *"A project that quietly bills £20/month
> while nobody visits gets deleted."*

---

## Platform characteristics as measured

| Platform | What it's good for | The catch |
|---|---|---|
| **Modal** | Per-second billing, scales to zero, hosts a full ASGI app *and* GPU functions on one platform. ~$30/mo free credits (≈50 T4-hours), 10 concurrent GPUs | ~6s cold start on the first request after idle. Vendor coupling |
| **Neon** (Postgres) | Auto-suspends when idle but **auto-resumes on the next connection in well under a second — no human in the loop** | Small free storage/compute ceiling |
| **Supabase** | Auth is the thing most worth not writing yourself (hashing, confirmation, token refresh, sessions), generous MAU tier | **Auto-pauses after 7 days of DB inactivity and needs a human to click Restore.** Also a limit on simultaneously *active* projects |
| **Cloudflare R2** | 10GB and **zero egress fees, permanently**. S3-compatible, so it's a boto3 swap | — |
| **Vercel** | Frontend, config pinnable in `vercel.json` | Preview URLs break CORS allowlists |
| **Railway** (retired) | — | Free tier is ~$1/mo credit, nowhere near four always-on services; the paid tier's included credit likely exceeded too |

**The decision that matters most for an occasionally-visited app: uptime, not cost.** Supabase's
7-day pause is *fatal* for a portfolio link — a reviewer finds a dead site and **only the owner can
revive it.** Neon auto-resumes with no human involved, which is why it beat the Postgres that was
already in the stack. Pick the auto-resuming option even when it adds a vendor.

## The platform can replace your infrastructure

**A durable, addressable, automatically-retried function call *is* a queue.** Once the API moved to
Modal, Redis + RQ were paying rent for a feature the platform gives away — `POST /upload` became a
`.spawn()` and returned.

Deleting the queue removed: a Redis instance, a worker process, a queue module, **and** the startup
recovery logic that existed only to clean up after it. That last one is the real prize — infra you
delete takes its maintenance code with it.

Cost: vendor coupling. Moving off means reintroducing a queue. Accepted deliberately, because the
alternative was paying for infrastructure to avoid a hypothetical migration.

## Things that are different under scale-to-zero

Containers start constantly, several at once, at unpredictable times. Three patterns that are fine
on an always-on box and **wrong** here:

- **Migrations on app startup.** Several containers starting together will race. Use a dedicated,
  deliberately-invoked one-shot migrate function. It also makes the deploy sequence legible.
- **Startup sweeps / reconciliation on boot.** A sweep that reset stale `processing` jobs to
  `pending` at startup would fire on **every cold start** and re-run jobs that were legitimately
  still in flight. Replace it with the platform's own `retries=N`; a job that exhausts them ending
  as `failed` is the honest outcome.
- **Keying "is the remote executor available?" on a credential being present.** Availability was
  keyed on `MODAL_TOKEN_ID` being set — but **a Modal container has no token; it is authenticated
  ambiently.** So the deployed API concluded Modal was unavailable and ran GPU inference locally,
  in an image that deliberately has no torch. Every upload failed with `No module named 'torch'`.
  Key on an API the platform provides (`modal.is_local()`), not on a secret's presence.

Relatedly: **keep the "where does this run" branch outside the worker function itself.** The same
function also runs *inside* the remote container, so an internal branch makes it dispatch to the
platform from within the platform — recursively.

## Config lives in the platform secret, not `.env`

- **Editing `.env` changes nothing in production.** Production config comes from the platform secret.
  Changing a value means recreating the secret **and** redeploying — in that order. The secret is
  read at container start, so the redeploy is required, not optional.
- **A warm container keeps serving the old value for a short while**, so a single check straight
  after deploying can read stale and look like the update failed. Poll for ~30s before concluding
  anything. This wastes a lot of time if you don't know it.
- **Functions resolved by name at runtime fail late.** If the dispatcher looks a function up by
  name, a rename or a missing deploy fails only **when a job actually runs** — not at API boot, not
  in CI, not in the health check.
- **Deploying the API and deploying the worker are separate steps.** Easy to do one and believe
  you're done.

## Storage: retention is not optional

Nothing deletes anything by default, so storage grows until the free tier fills — and **it surfaces
as an upload error that reads like a bug rather than a quota.** Budget this in from the start.

What worked: a daily cron expires jobs past a window, deletes the audio, and marks the record
`expired`. Notes on the design:

- **Keep the record, delete the payload.** A stale link then says *"this existed, its audio is
  gone"* instead of 404-ing indistinguishably from a bug.
- **Store derived artefacts in the cheap format.** 256kbps MP3 instead of WAV cut per-job storage to
  ~20–60MB and made waveforms load ~10× faster. That is what makes a 1GB tier viable at all.
- **Exempt special records automatically, not by a second config list.** The public demo job is
  exempt *because it's the demo job* — requiring someone to also list it in
  `RETENTION_EXEMPT_JOB_IDS` would be easy to get wrong, and getting it wrong deletes the public
  demo's audio two weeks later with no warning.
- **Dry-run any destructive sweep against real data before enabling it.** There are no backups on a
  free plan. Recovery is genuinely none.
- **Beware convenient coupling.** The sweep also kept the Supabase project alive by touching it
  daily. Handy — but now if the sweep breaks, storage stops being reclaimed **and** the project
  eventually pauses. Two failures behind one silent cause.
- **Private buckets + short-lived signed URLs** (1 hour) rather than public objects. Costs a round
  trip before playback; a leaked URL expires and bulk enumeration is impossible.

## Supabase specifics that cost real time

- **Use the project URL (`https://<ref>.supabase.co`), not the dashboard's REST URL.** Pasting the
  REST one makes the client build `/rest/v1/auth/v1/token`, which 404s — and an auth layer turns
  that into a generic **`401 Invalid or expired token`**, so it reads as bad credentials. Strip
  `/rest/v1`, `/auth/v1`, `/storage/v1` suffixes defensively.
- **Buckets do not exist until someone creates them**, and nothing creates them on demand. Fresh
  project = uploads fail.
- **Email confirmation is on by default**, so a new account is unusable until the link is clicked.
  The built-in SMTP is rate-limited to a couple of messages an hour and frequently lands in spam.
  Confirm directly via the admin API when the mail never arrives.
- A paused project **cannot be restored if another project holds the active-project slot.**

## Operational hygiene

- **One mailbox for all service accounts.** Pause notices, quota alerts and billing warnings scatter
  across a personal inbox with no shared heading — and the pause notice is exactly the email worth
  not missing. A Gmail `+alias` (`you+projectname@gmail.com`) works on every one of these platforms
  immediately and filters cleanly. Zero setup.
- **Write down which value comes from which dashboard**, once, in a runbook. See
  `patterns/project-documentation.md`.
- Treat a dormant project as untrusted: this stack broke after four months idle in three separate
  ways (a paused project, a deleted project, and a library's changed default driver).

## Related

- `stacks/python-fastapi.md` · `stacks/postgres-and-data.md` · `stacks/react-vite-typescript.md`
- `patterns/portfolio-project-criteria.md` — why uptime is the real requirement
- `projects/tracksplit-music-tool.md`
