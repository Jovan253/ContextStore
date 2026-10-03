# Debugging playbook

Generalised from the bugs recorded across KubePlayground, TrackSplit and Dig & Sell. Read this when
you are about to guess.

---

## 1. A successful deploy only proves the image built

The sharpest line in any of these repos. Several TrackSplit bugs **passed CI, passed a health check,
and failed on the first real file**:

- the dispatcher picked the wrong executor, so inference ran in an image with no torch
- a transitive dependency was missing, surfacing at import rather than at build
- an auth host was subtly wrong, reported as bad credentials

**So: get one genuine end-to-end request through before believing anything works.** A green
pipeline, a 200 from `/health`, and a rendered template are three different claims, none of which is
"it works".

## 2. Make the system tell you, don't guess at it

The OIDC trust-policy failure (`sts:AssumeRoleWithWebIdentity` not authorized, error gives no
reason) was solved by **having the workflow request its own token and print the `sub` claim** —
revealing that GitHub injects numeric IDs into every name.

Generalise it: when an opaque authz/identity error has many candidate causes, **find a way to print
what the system actually sees** instead of permuting what you send it. Same reasoning behind
rendering a Kubernetes object "as written" vs "as stored" to understand admission control.

Caveat learned alongside: print the specific claim, never the whole token — **a token is a bearer
credential and must never reach a log.**

## 3. Two wrong guesses means change the loop, not the guess

A Roblox tool's grip rotation got two blind guesses from code — Z axis, then X axis. Both wrong, and
X made it **visibly worse**. The commit that fixed it is literally titled *"stop guessing blind at
rotation"*: revert to a known baseline, then have the human **live-tune it in the engine's own
console** and report the value, which took one round trip instead of N.

**If the feedback signal lives in a human's eyes, put the human in the loop rather than iterating
blind.** This applies to anything spatial, visual, or aesthetic — rotations, offsets, timings,
colour, difficulty.

## 4. Fixing a real bug does not mean you fixed *the* bug

Mining was broken with a tool equipped. `CanQuery` defaulting to true *was* a genuine bug blocking
the raycast — fixed it, still broken. The actual cause was that **default controls stop evaluating
ClickDetectors entirely once any Tool is equipped**, which made the whole architecture impossible.

**After a fix, re-verify the original symptom specifically.** "I found a bug" and "I found the bug"
are different claims. Also note the fix that *reveals* the second bug is still worth keeping.

## 5. Ask which layer refused you

A 403 from `kubectl` on EKS could be **AWS IAM** (authentication, fetching the endpoint) or
**Kubernetes RBAC** (authorisation, via an access entry). They're separate systems with separate
configuration. Naming the two candidate layers before testing either is faster than poking at one.

Generally: for any request crossing a boundary, enumerate the layers that could have rejected it
*before* you start changing things.

## 6. Averages hide sub-second damage

A container averaging 142m CPU against a 500m limit was throttled in **7.5% of scheduling windows**,
because **CFS quota is enforced per 100ms period, not as an average**. Dashboards averaging over
30–60s showed no pressure at all. `nr_throttled` was the only place it appeared.

**When a metric looks fine but behaviour doesn't, check the enforcement window, not the average.**
This is the classic cause of p99 latency spikes with no visible CPU pressure.

## 7. Prefer loud failures; distrust silent ones

**CPU limits degrade you silently, memory limits kill you loudly.** Given the choice, take the loud
failure — an OOMKill with exit code 137 is diagnosable, a 7% throttle is not.

Catalogue of silent failures from these repos, all of which produced *no error anywhere*:

| Silent failure | What you see instead |
|---|---|
| Node served a cached image layer for a rebuilt tag | Old code running, deploy "succeeded" |
| Ingress rules with no controller installed | Nothing routes; no event, no error |
| `subPath` ConfigMap mount never updating | Stale config forever |
| A JSON-column `isnot(None)` matching every row | A sweep that re-processes everything, forever |
| `react-force-graph` mutating link endpoints | A filter that works on first render then stops |
| Invisible `ParticleEmitter` with no `Texture` | Nothing renders, no warning |
| DataStore load failure treated as "new player" | Progress silently overwritten |

**When something is "not working" with no error, suspect a default you didn't set** rather than a
line you wrote.

## 8. Error messages lie — specifically, they report the wrong layer

- A wrong auth *host* 404'd, and the auth layer turned that into **`401 Invalid or expired token`**.
  So the message says "bad credentials" and the cause is a URL.
- `containerPort` set wrong repointed a named port, so probes hit a closed port and the container
  CrashLoopBackOff'd — **while the application was completely fine.** The message blames the app.
- `sts:AssumeRoleWithWebIdentity` not authorized — says nothing about which claim mismatched.

**Treat the error's *category* as a hypothesis, not a fact.** Ask what else could produce this exact
message.

## 9. The bug is invisible until the second instance exists

Checkpoint state keyed only by player, not by player *and* zone, was correct-looking with one
parkour zone and wrong with two. Similarly, event field selectors need **both** kind and name,
because a name is only unique within a kind — invisible until a Deployment and a Service share a
name, which happened.

**When you add the second instance of anything, go looking for state that assumed there was one.**

## 10. First moves, by platform

| Platform | First move |
|---|---|
| Kubernetes | `kubectl describe <obj>` → the **Events** section |
| EKS 403 | Which layer — AWS IAM or Kubernetes RBAC? |
| Serverless function | The platform's per-invocation logs; was the *other* app even deployed? |
| Config change had no effect | Is production config coming from a secret, not `.env`? Did you redeploy? Is a warm container serving stale? |
| Frontend works in prod but not dev (or vice versa) | StrictMode, or a build-time-baked env var |
| Roblox | Playtest it; almost nothing here was found by reading |
| Dormant project | Assume the platforms changed under you, not the code |

## Finally: write it down as a symptom

Every entry in this file exists because somebody recorded the symptom in the words it first appeared
in. **Record the symptom, not just the fix** — future-you searches by symptom. See
`patterns/project-documentation.md`, and add to `INDEX.md`'s symptom table.
