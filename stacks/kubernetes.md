# Kubernetes

Learned the hard way in **KubePlayground** (Aug–Sep 2026), an app deployed *into* the cluster it
observes. Every item below broke something real first.

---

## Probes

Four mistakes, each of which applied cleanly and broke something else:

- **`exec` takes a COMMAND, not a URL path.** `/api/health` is not a file on disk.
- **A probe talks straight to the Pod IP.** It never goes through the Service, so the port is
  `containerPort` (e.g. 8000), never the Service's `port` (e.g. 80).
- **`containerPort` binds nothing — it is documentation.** Setting it to 80 did not move the
  listener, but it *did* repoint the named port `http`, so every probe hit a closed port and the
  container CrashLoopBackOff'd while the app itself was perfectly healthy. This is a nasty one:
  the symptom points at the app, the cause is in the manifest.
- **With a `startupProbe` present, liveness and readiness do not run until it succeeds**, which
  makes `initialDelaySeconds` on them redundant. That is the entire reason startup probes exist:
  a generous boot budget *and* fast failure detection afterwards, instead of trading one for the
  other via a large `initialDelaySeconds`.

When testing probes, **break them separately**. If liveness and readiness are configured
identically they always fail together, so you never see the difference: readiness drops the Pod
out of the EndpointSlice and leaves it running; liveness kills and restarts the container.

## Resource requests and limits

**Set them from measurement, not guesswork.** Read the container's own cgroup rather than
estimating:

```
/sys/fs/cgroup/memory.current
/sys/fs/cgroup/cpu.stat
```

Measured for a small FastAPI app: 6m CPU / 81 MiB idle, **142m** CPU / 86–90 MiB under ~3 req/s.

**The finding worth keeping.** At a `500m` CPU limit while averaging 142m — 28% of the allowance —
the container was **still throttled in 11 of 147 scheduling windows (7.5%)**, losing 0.26s to
forced idling in 16 seconds.

Why: **CFS quota is enforced per 100ms period, not as an average.** `500m` means 50ms of CPU per
100ms window. A bursty app — request lands, serialises a few hundred objects in one tight burst,
then idles — will blow through 50ms inside a single window and be stopped dead until the next one.

This is the classic production misdiagnosis: p99 latency spikes with no CPU pressure visible
anywhere, because dashboards average over 30–60s and the damage happens in 100ms slices.
`nr_throttled` in `cpu.stat` is the only place it shows.

**The asymmetry to remember: CPU limits degrade you silently, memory limits kill you loudly.**
Hence the defensible position — set a CPU *request* and consider omitting the CPU *limit* entirely
for latency-sensitive services, while memory limits stay mandatory because the failure mode there
is an OOMKill (exit code 137), not a stall. Prefer the loud failure.

**QoS `Guaranteed` needs requests == limits for both CPU and memory on every container.** Matching
only CPU buys nothing; you stay `Burstable`.

## RBAC

- **A fresh ServiceAccount can do nothing.** The Pod will be `1/1 Running` and every single API
  call 403s. Seeing this baseline once is worth more than reading about it.
- **RBAC is additive — there are no deny rules.** Effective permissions are the union of every
  binding naming a subject. An absent rule protects nothing; a wrongly-granted one cannot be
  subtracted elsewhere.
- **Subresources are separate RBAC targets.** `pods/log` and `deployments/scale` must be granted
  explicitly, in addition to `pods` and `deployments`.
- **API group matters as much as the resource.** Ingresses live in `networking.k8s.io`, neither
  core nor `apps`, so they need their own rule. Same trap for anything outside `v1`/`apps/v1`.
- Prove it works by *removing* a verb and watching the 403 arrive.
- `kubectl auth can-i --list --as=system:serviceaccount:<ns>:<name>` is the fast check.
- **Escalation prevention:** you cannot create a Role granting permissions you do not already
  hold. This is why a namespace-scoped CI identity physically cannot install a chart that creates
  a ClusterRole — see `stacks/aws-terraform-and-oidc.md` for the production split that solves it.
- **No cross-namespace references.** ConfigMaps, Secrets and Ingress backends can only be named
  from the referencing object's own namespace. This stops being trivia and starts shaping design
  the moment you have one Ingress per namespace.

## Services, Ingress and how traffic actually arrives

- `port` vs `targetPort` — the Service's port and the container's port, the first time they differ
  is the moment this clicks.
- ClusterIP is **virtual**: there is no interface holding it, just kube-proxy rules. Pod CIDR and
  Service CIDR are different ranges.
- **EndpointSlices auto-populate from label matches.** Empty endpoints = a selector bug, basically
  always.
- A **Service's selector is a plain map**, unlike a Deployment's `selector.matchLabels`.
- Cluster DNS is `<service>.<namespace>.svc.cluster.local`; short names work via the search domain.
- **Ingress *resource* vs Ingress *controller*.** The resource is rules in etcd; the controller is
  the workload that acts on them. **Rules with no controller silently route nothing** — no error,
  no event, just nothing.
- `pathType` is required: `Prefix` / `Exact` / `ImplementationSpecific`.
- **A missing `apiVersion` is one of the few things Kubernetes rejects outright** rather than
  accepting silently. Most other mistakes are accepted and simply don't work.

### Host-based vs path-based routing

- **Host-based** (`site-a.localhost`, `site-b.localhost`) — browsers resolve `*.localhost` to
  127.0.0.1 with no hosts-file edit, and it needs no path rewriting. Cleanest for local demos.
- **Path-based** (`/site-a`) looks simpler but **nginx forwards the full path**, so the backend
  receives `/site-a/index.html` and 404s. It needs a capture-group path plus an annotation:

  ```yaml
  path: /site-a(/|$)(.*)
  pathType: ImplementationSpecific
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  ```

  Note `pathType` changes from `Prefix` to `ImplementationSpecific` — **regex paths are not part of
  the Ingress spec, they are an nginx extension**, which is exactly why `rewrite-target` is a
  controller-specific annotation and not a spec field. That is the portability cost, and the reason
  path-based routing has its reputation.
- **Precedence:** a rule with a specific `host:` beats a host-less catch-all. ingress-nginx sorts
  paths **longest-first**, so `/site-a` wins over `/`. Verify rather than assume.
- An Ingress controller's real purpose is **N sites behind ONE load balancer**. A single-app setup
  never demonstrates it. On a cloud provider this is also the cost argument: three `LoadBalancer`
  Services is three load balancers (~$17/mo each) to demonstrate worse architecture.

## Ownership, and the relationship nothing stores

`Pod → ReplicaSet → Deployment` is recorded as data in `ownerReferences`, so you can walk it.

But **a Service finds its Pods by label selector, so nothing stores that relationship.** To build
an `Ingress → Service → Deployment` link you have to match `service.spec.selector` against the
Deployment's own selector and *infer* it.

**Selectors are a query, not a link.** This is the single most clarifying sentence about
Kubernetes' object model.

## Deployments and rollouts

- **A Deployment's `spec.selector` is IMMUTABLE.** Anything version-ish inside it makes every
  future upgrade fail. See `stacks/helm.md` for the label-helper split this forces.
- `pod-template-hash` is what distinguishes the two ReplicaSets during a rolling update.
- **Imperative `kubectl scale` causes drift from the file.** The file is the source of truth.
- **The rolling-update safety net, observed accidentally:** a Deployment was bumped to an image
  tag that had not been built yet, so the new ReplicaSet sat in `ImagePullBackOff` for 22 hours —
  **and the site stayed up the whole time.** The rollout never completed, so the old ReplicaSet was
  never scaled down. A far better demonstration than a deliberate test. Once the image existed,
  deleting the stuck Pod was faster than waiting out the exponential backoff.

## Images and tags

**Treat image tags as immutable. Harder than you think, and the failure is silent.**

On a local Docker Desktop cluster (which is now **kind** under the hood, `containerd` runtime, not
the old kubeadm-on-the-host setup), a locally-built tag *is* usable — but by a different mechanism
than the old Rancher Desktop behaviour of sharing one Docker daemon. The kubelet pulls over the
registry protocol from Docker Desktop's registry mirror, which fronts your local image store. It
reports `Pulling image` and `Successfully pulled` even under `imagePullPolicy: IfNotPresent`.

**Consequence: the node CACHES what it pulls.** Rebuild the same tag and `IfNotPresent` will not
re-pull it — **the Pod serves stale layers with no error anywhere.** Bump the tag on every build.

A real registry becomes mandatory the moment the node is not your own laptop.

## ConfigMaps and Secrets

- **Mounted ConfigMap volumes update live; env vars are frozen at container start** — you need
  `kubectl rollout restart` to pick them up.
- **`subPath` mounts never update.** Straight gotcha, no workaround.
- ConfigMaps are plaintext; **Secrets are base64-encoded, not encrypted.**

## Events and debugging

- **`kubectl describe` → the Events section is the first debugging move.** Always.
- Events are **separate objects in the core group** and they carry no useful labels, so you filter
  them with a **field selector**, not a label selector:
  `involvedObject.kind=...,involvedObject.name=...` — comma-separated, and you need both because a
  name is only unique within a kind (a Deployment and a Service can share a name). Needs its own
  `events` RBAC grant.

## Core mental model

**`spec` is your intent; `status` is system-written reality.** Everything else follows from this.
A good way to make it concrete: render a live object twice — *as written* and *as stored* — and
list what the server added or stripped (`status`, `managedFields`, `uid`, `resourceVersion`,
`ownerReferences`, the kubelet-injected ServiceAccount token volume). The removal is the lesson.

Related: **admission control means the server can modify what you submit** (e.g. the
`kubernetes.io/metadata.name` label on namespaces).

Validation habits worth having: `--dry-run=client` vs `--dry-run=server` vs `kubectl diff`.
Also `kubectl api-resources --namespaced=false` to see what is cluster-scoped.

## Safety rule — multiple contexts

When a kubeconfig holds both a sandbox and something real (an employer's AKS cluster, or your own
EKS), **pass `--context` explicitly on every mutating command** (`apply`, `delete`, `scale`,
`edit`), or verify first:

```powershell
kubectl config current-context
```

Note that `aws eks update-kubeconfig` makes the new cluster the **current** context, which is
precisely the hazard. Rename it and switch back so an unqualified command hits the sandbox.

## Related

- `stacks/helm.md` — packaging these manifests
- `stacks/aws-terraform-and-oidc.md` — the same app on EKS
- `projects/kubeplayground.md` — where all of this came from, and what's still open
