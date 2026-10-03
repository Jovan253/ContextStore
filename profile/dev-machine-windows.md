# Dev machine — Windows

**Verify before relying on any version number here.** Versions drift; the *quirks* are the durable
part of this file.

Last checked: **2026-10-03**.

---

## The basics

- **Windows.** This machine reported Windows 10 Home 10.0.19045 on 2026-10-03; KubePlayground's notes
  (Aug 2026) say Windows 11 — so **he has more than one machine, or it changed.** Confirm rather than
  assume.
- **PowerShell is the primary shell.** Write PowerShell, not bash, for anything he'll run.
- **Repos live in `C:\Users\Jovan\Documents\projects`.** A couple also under
  `C:\Users\Jovan\source\repos` (Visual Studio projects).
- **`gh` CLI is NOT installed** (confirmed 2026-10-03). Use the GitHub API via `WebFetch`, or plain
  `git`. `git` is at `C:\Program Files\Git\cmd\git.exe`.
- There is no global `~/.claude/CLAUDE.md` as of 2026-10-03.

## Runtime versions seen

Treat as a range, not a fact — they differ per project and over time.

| Runtime | Seen |
|---|---|
| Node | 20.11.0 → 20.18.0 (TrackSplit, at different dates) |
| Python | 3.11.8 on PATH; 3.12 targeted by some projects |
| Docker | 29.7.2 |
| Kubernetes (local) | v1.36.1 via Docker Desktop |
| helm | v4.2.4 |

**Node version is load-bearing:** Vite 8+ needs Node 20.19+, which is why TrackSplit is pinned to
Vite 5.x — while Mythos runs Vite 8 happily. **Check the installed Node version and the project's
lockfile before recommending an upgrade.** See `stacks/react-vite-typescript.md`.

## Local Kubernetes

**Docker Desktop's built-in Kubernetes**, context `docker-desktop`, single node
`desktop-control-plane`.

Important: Docker Desktop now provisions that cluster with **kind** under the hood
(`kindest/node:...`), not the old kubeadm-on-the-host setup — so the container runtime is
**containerd**, not docker. It also runs a registry mirror (`desktop-containerd-registry-mirror`) and
`desktop-cloud-provider-kind` for LoadBalancer Services.

Consequences, in full, in `stacks/kubernetes.md`. The two that bite:

- **No ingress controller ships with it.** (k3s bundled Traefik; Docker Desktop bundles nothing.)
  ingress-nginx was installed manually, which creates the `nginx` IngressClass — so manifests must
  say `ingressClassName: nginx`.
- **The node caches images it pulls**, so rebuilding the same tag with `imagePullPolicy: IfNotPresent`
  silently serves stale layers. **Bump the tag every build.**

`LoadBalancer` Services work and Docker Desktop forwards `localhost:80` to them, so `http://localhost/`
is the entry point. A bare `http://localhost/` with no matching Ingress rule returning nginx's 404
default backend is a **healthy** answer, not a failure.

StorageClasses are `standard` (default) and `hostpath`, both on the `rancher.io/local-path` provisioner.

**History worth knowing:** Rancher Desktop + k3s + WSL2 were the original setup and are **gone** (as of
2026-08-30). Notes in older project docs referring to k3s, Traefik or `rancher-desktop` are stale.

## Windows-specific gotchas

The recurring ones, all of which have cost real time:

- **Closing a terminal does not kill `uvicorn`.** A stale process holds the port while your new server
  idles — requests hit the old code and the symptom looks like an app bug.

  ```powershell
  Get-NetTCPConnection -LocalPort 8000 -State Listen | ForEach-Object { Stop-Process -Id $_.OwningProcess -Force }
  ```

  Nuclear option: `Get-Process python | Stop-Process -Force`.

- **RQ needs `SimpleWorker` on Windows** to avoid fork issues.
- **`torchaudio` 2.11 dropped soundfile on Windows** — hence the `torch==2.5.1`/`torchaudio==2.5.1`
  pin in TrackSplit.
- **ffmpeg is a separate system install**, not a pip dependency: `winget install Gyan.FFmpeg`.
  `pydub` fails at runtime without it, with a confusing warning.
- **Virtualenv executables are invoked by path** in his runbooks, e.g. `.venv\Scripts\modal.exe deploy`
  rather than relying on activation.

## Multi-context safety habit

His kubeconfig has at times held an **employer AKS context** alongside the local sandbox. The standing
rule, kept even when that context is absent:

> Every mutating command (`apply`, `delete`, `scale`, `edit`) passes `--context docker-desktop`
> explicitly, or the active context is verified first with `kubectl config current-context`.

Note `aws eks update-kubeconfig` makes EKS the **current** context — rename it and switch back.

## Related

- `stacks/kubernetes.md` · `stacks/python-fastapi.md` · `stacks/react-vite-typescript.md`
