# Python, FastAPI and packaging

Mostly from **TrackSplit** (repo `Music-Tool`), a FastAPI API with GPU audio separation, developed
on Windows.

---

## Windows development

- **Closing a terminal does not kill uvicorn.** A stale process keeps holding port 8000 while your
  new server idles — so requests go to the old code and the symptom is *"uploads silently do
  nothing"*, which reads like an application bug. Always clear the port before restarting:

  ```powershell
  Get-NetTCPConnection -LocalPort 8000 -State Listen | ForEach-Object { Stop-Process -Id $_.OwningProcess -Force }
  ```

  If it respawns or the PID won't resolve: `Get-Process python | Stop-Process -Force`.

- **RQ workers need `SimpleWorker` on Windows** to avoid fork issues:
  `rq worker default --worker-class rq.SimpleWorker`

- **`uvicorn --reload` may kill a long-running in-process background task mid-run.** If you're
  running real work as a FastAPI background task during development, expect it to die on the next
  file save.

- **ffmpeg is a separate system install.** `pydub` needs it for mixdown/transcode, and without it
  export fails at runtime with a confusing pydub warning rather than a clear error.
  `winget install Gyan.FFmpeg`.

## Packaging

- **Build backend: always `setuptools.build_meta`**, never `setuptools.backends.legacy:build`.
- **Use optional extras to keep the core install light.** This mattered a lot here:

  | Install | Gets you |
  |---|---|
  | `pip install -e .` | API, storage, cloud client. **No torch.** |
  | `pip install -e ".[dev]"` | the above plus pytest — normal development |
  | `pip install -e ".[local-separation]"` | adds torch + demucs (~2.5GB), only to run inference locally |

  A 2.5GB dependency that 90% of your work doesn't need should not be in the default install, and
  keeping it out also keeps it out of the deployed image.

## ML / heavy dependencies

- **Neither torch nor demucs declares numpy in its metadata.** It arrives transitively or not at
  all — so the image builds fine and then **fails at import with `No module named 'numpy'`**.
  Declare the transitive deps you actually rely on.
- **Pin torch and torchaudio together.** `torch==2.5.1` / `torchaudio==2.5.1` here, because
  torchaudio 2.11 dropped soundfile support on Windows.
- Dispatch decisions about *where* heavy inference runs belong **outside** the worker function —
  see `stacks/free-tier-hosting.md`, where keying that decision on the wrong signal caused the
  deployed API to try running Demucs in an image that deliberately had no torch.

## API design notes that came from bugs

- **`update_job(error=None)` meaning "leave unchanged" is a trap.** It meant an error could never
  be *cleared*, so a successfully retried job displayed `done` next to a stale error message. If
  `None` means "don't touch", you need a separate sentinel for "set to null" — or a different
  signature.
- **Don't accept an identifier from the request on an unauthenticated route.** If a public demo
  endpoint took a job id as a parameter, anyone could read anyone's data. Take it from
  configuration, so **there is no request shape that reaches another user's content.**
- Catch storage/SDK exceptions inside routes rather than letting them bubble, or the error response
  **loses its CORS headers** and the browser reports an opaque CORS failure instead of your actual
  500.

## The lesson that generalises

> **A successful deploy only proves the image built.**

Several bugs here passed CI, passed a health check, and then failed on the first real file. Build
a path for one genuine end-to-end request before believing anything. See
`patterns/debugging-playbook.md`.

## Related

- `stacks/free-tier-hosting.md` — deploying FastAPI serverless, config via platform secrets
- `stacks/postgres-and-data.md` — SQLAlchemy and Alembic
- `profile/dev-machine-windows.md`
