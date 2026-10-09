# Index — start here

Routing table for the context store. Match on **symptom** or **stack**, then fetch the file.

Fetch base (append the path from the tables below):

```
https://raw.githubusercontent.com/Jovan253/ContextStore/main/
```

Example: `https://raw.githubusercontent.com/Jovan253/ContextStore/main/stacks/kubernetes.md`

---

## By stack

| Working with… | Fetch |
|---|---|
| Kubernetes — probes, RBAC, Services, Ingress, resources, images | `stacks/kubernetes.md` |
| Helm charts, templating, releases | `stacks/helm.md` |
| AWS, Terraform, GitHub OIDC, EKS, ECR, cost | `stacks/aws-terraform-and-oidc.md` |
| GitHub Actions pipelines | `stacks/aws-terraform-and-oidc.md` (OIDC auth) + `stacks/helm.md` (deploy step) |
| Python, FastAPI, uvicorn, packaging, ML deps | `stacks/python-fastapi.md` |
| React, Vite, TypeScript, StrictMode, Vercel | `stacks/react-vite-typescript.md` |
| Postgres, SQLAlchemy, Alembic | `stacks/postgres-and-data.md` |
| Free-tier / serverless hosting (Modal, Supabase, Neon, R2, Railway, Vercel) | `stacks/free-tier-hosting.md` |
| Roblox, Luau, Studio MCP, Rojo, Blender → Studio | `stacks/roblox-luau.md` |

## By situation

| Situation | Fetch |
|---|---|
| Starting a new project — what docs to create | `patterns/project-documentation.md` |
| Starting a new Roblox game — tooling setup and workflow | `stacks/roblox-luau.md` (Recommended setup) |
| Stuck on an opaque error; about to guess | `patterns/debugging-playbook.md` |
| Building a feature in rounds with human feedback | `patterns/staged-build-and-playtest.md` |
| Deciding whether something is worth building / shipping publicly | `patterns/portfolio-project-criteria.md` |
| First time working with Jovan, or unsure how much to decide alone | `profile/working-agreement.md` |
| Need to know the machine's tooling, versions, OS quirks | `profile/dev-machine-windows.md` |
| Need to calibrate explanation depth to his experience | `profile/about-jovan.md` |

## By symptom

Highest-value table in the store. These are all mistakes that have already cost real debugging time.

| Symptom | Likely cause | Fetch |
|---|---|---|
| `sts:AssumeRoleWithWebIdentity` not authorized, trust policy looks correct | GitHub OIDC `sub` carries numeric account/repo IDs; `StringEquals` never matches the tutorial form | `stacks/aws-terraform-and-oidc.md` |
| `kubectl` 403 on EKS but IAM looks right | Two permission systems; an access entry alone grants nothing | `stacks/aws-terraform-and-oidc.md` |
| Pod `1/1 Running` but every API call 403s | Fresh ServiceAccount has no permissions at all | `stacks/kubernetes.md` |
| Container CrashLoopBackOff but the app runs fine locally | `containerPort` binds nothing; changing it repoints named ports and breaks probes | `stacks/kubernetes.md` |
| p99 latency spikes with no CPU pressure on any dashboard | CFS quota is enforced per 100ms window, not as an average | `stacks/kubernetes.md` |
| Service has no endpoints | Selector doesn't match pod labels; EndpointSlice is populated by label match | `stacks/kubernetes.md` |
| Ingress exists, routes nowhere, no error | Rules with no controller silently do nothing | `stacks/kubernetes.md` |
| Rebuilt the image, pod still serves old code, no error anywhere | Node cached the tag; `IfNotPresent` won't re-pull. Treat tags as immutable | `stacks/kubernetes.md` |
| ConfigMap change not picked up | env vars freeze at container start; `subPath` mounts never update | `stacks/kubernetes.md` |
| `helm upgrade` fails after a chart version bump | Version label leaked into the immutable Deployment selector | `stacks/helm.md` |
| `helm install` fails on ownership | Helm will not adopt objects `kubectl apply` created | `stacks/helm.md` |
| `helm lint` passes, cluster rejects the YAML | lint checks structure only; `helm template` is what renders | `stacks/helm.md` |
| Uploads/requests silently do nothing on Windows | Stale uvicorn still holding the port; closing a terminal doesn't kill it | `stacks/python-fastapi.md` |
| Image builds, then fails at import with a missing common package | ML packages don't declare all transitive deps in metadata | `stacks/python-fastapi.md` |
| Everything passed CI and the health check, then failed on the first real input | A successful deploy only proves the image built | `patterns/debugging-playbook.md` |
| Sign-in returns a generic 401 with valid credentials | Auth host is the REST URL, not the project URL; the 404 is reported as 401 | `stacks/free-tier-hosting.md` |
| Config change has no effect in production | Production config lives in the platform secret, not `.env`; needs recreate *and* redeploy | `stacks/free-tier-hosting.md` |
| App wouldn't start after months dormant | SQLAlchemy 2.1 changed the default `postgresql://` driver | `stacks/postgres-and-data.md` |
| A sweep/filter matches every row it should skip | SQLAlchemy `JSON` stores Python `None` as JSON `null`, not SQL `NULL` | `stacks/postgres-and-data.md` |
| Storage tier filled up, surfacing as a confusing write error | Nothing ever deleted anything; retention isn't optional | `stacks/free-tier-hosting.md` |
| React effect fires twice / stale async callback wins | StrictMode double-invoke; counters incremented by callbacks break | `stacks/react-vite-typescript.md` |
| Graph/filter works on first render, breaks after | `react-force-graph` mutates `link.source`/`target` into node objects | `stacks/react-vite-typescript.md` |
| `VITE_` var changed in a dashboard, bundle still has the old value | Baked in at build time; needs a redeploy, not a save | `stacks/react-vite-typescript.md` |
| Vercel monorepo first build fails | Root Directory is the one setting `vercel.json` can't supply | `stacks/react-vite-typescript.md` |
| Mining/clicking stops working the moment a Tool is equipped | Default controls stop evaluating ClickDetector entirely with any Tool equipped | `stacks/roblox-luau.md` |
| Clicks don't reach the object behind a held tool | `CanQuery` defaults true independent of `CanCollide` | `stacks/roblox-luau.md` |
| ParticleEmitter renders nothing | It needs a `Texture`; there's no usable default | `stacks/roblox-luau.md` |
| UI list items all stack at (0,0), nothing scrolls | `ClearAllChildren` destroyed the `UIListLayout` too — it's a child | `stacks/roblox-luau.md` |
| A timed reveal animation spoils itself early | PlayerData replicates on grant, well before the animation finishes | `stacks/roblox-luau.md` |
| Player progress silently reset | DataStore load failure treated as "new player", then saved over | `stacks/roblox-luau.md` |
| Teleported character slides off its platform and falls; model lands offset after `PivotTo` | `WorldPivot` is ignored once a `PrimaryPart` is set — use `PrimaryPart.PivotOffset` | `stacks/roblox-luau.md` |
| Module state is `nil`/fresh when inspected via Studio MCP `execute_luau` | It runs in its own VM; `require` returns a new module instance | `stacks/roblox-luau.md` |
| `claude mcp add` server times out (30000ms); registered args show `C:/` instead of `/c` | Git Bash (MSYS) path conversion — `MSYS_NO_PATHCONV=1` | `profile/dev-machine-windows.md` |
| Blender MCP add-on install: "No Blender addons directories found" / `UnicodeEncodeError` | Blender never launched yet; cp1252 console | `stacks/roblox-luau.md` (Blender) |
| `winget install` appears hung | Waiting on a UAC prompt | `profile/dev-machine-windows.md` |
| Blind-guessing a rotation/offset axis, each guess a code round-trip | Change the loop: live-tune in the engine console, bake the result | `patterns/debugging-playbook.md` |

## Projects

Per-repo pages: what it is, its stack, where its own docs live, and what state it was last left in.

| Project | Fetch |
|---|---|
| KubePlayground — live cluster visualiser, KCNA vehicle | `projects/kubeplayground.md` |
| TrackSplit (repo `Music-Tool`) — GPU stem separation | `projects/tracksplit-music-tool.md` |
| Dig & Sell (repo `RobloxAppDigSell`) — published Roblox simulator | `projects/roblox-dig-and-sell.md` |
| Clueless: Lava Rising (repo `Roblox-Clueless`) — semantic word-guessing Roblox game, Studio-first via MCP | `projects/roblox-clueless.md` |
| Mythos — mythology knowledge graph | `projects/mythos.md` |
| portfolio — personal site | `projects/portfolio.md` |
| sudoku-solver — visual solver | `projects/sudoku-solver.md` |
