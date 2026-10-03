# Portfolio project criteria

Most of Jovan's personal projects are **portfolio pieces as well as learning vehicles**, and that
dual purpose shows up in the architecture. These are the criteria his own repos keep arriving at —
apply them when scoping or reviewing a personal project.

---

## Cost is an architectural constraint, not a footnote

> *"This is a portfolio piece and a personal tool. A project that quietly bills £20/month while
> nobody visits gets deleted."*

Consequences that actually followed from that sentence:

- Scale-to-zero platforms over always-on services, even at the cost of cold starts
- EKS built to be **raised and destroyed on demand**, with `destroy` as a triggerable workflow —
  because the control plane alone is ~$73/month idle
- *"The pipeline stands the whole environment up from nothing"* reframed as the **better story**
  rather than a compromise
- Path-based routing on one ingress controller instead of three load balancers — *"roughly $50/mo to
  demonstrate worse architecture"*

**Design for $0 idle, and let that be the interesting constraint rather than a limitation you
apologise for.**

## Uptime matters more than cost, and for the same reason

A portfolio link gets clicked rarely and unpredictably. So:

- **A platform that pauses and needs a human to un-pause is disqualifying.** A reviewer finds a dead
  site and **only you can revive it** — you won't know it happened.
- Prefer auto-resume (sub-second, no human) over auto-pause-and-click, even if it means adding a
  vendor. See `stacks/free-tier-hosting.md`.
- Assume any dormant project is broken until proven otherwise. TrackSplit broke in three unrelated
  ways after four months idle.

## Nobody will sign up

The highest-leverage item identified in TrackSplit, and the reasoning is worth quoting:

> *"Nobody evaluating this will sign up: the sign-in wall, the email confirmation step and the
> free-tier limits all stand between a visitor and seeing anything work. A public page carrying one
> pre-separated track whose stems can be played and mixed without logging in is what makes the link
> worth sending."*

If your project has auth, **a public demo is not polish — it's the feature that makes the project
legible at all.** Three design rules that came out of building one:

- **The demo's record id comes from configuration, never from the request.** Otherwise anyone can
  read anyone's data. *"Do not 'helpfully' add a parameter."*
- **Exempt the demo's data from retention automatically**, by virtue of being the demo — not via a
  second config list someone has to remember. Forgetting deletes the demo's content weeks later with
  no warning.
- **Hide actions that require auth** rather than letting them 401 in front of a visitor.

## The image is what gets judged

> *"Nobody clones a repo; the image is what they judge."*

- A demo GIF / screenshot is the README's most important asset.
- **Record it against the real cloud deployment, not localhost.** Identical UI, but the version
  string, the node name and the registry path all prove it's real.
- *"For a portfolio piece the look carries disproportionate weight, since it is the first and
  sometimes only thing a visitor judges."* Treat a UI pass and a UX pass as one combined piece of
  work, since they touch the same components.
- Mermaid diagrams in the README over prose — see `patterns/project-documentation.md`.

## Deliberately pick the gap, not the strength

KubePlayground's M9 chose **AWS + GitHub Actions rather than Azure + GitLab**, explicitly because
*"those are already day-job skills, and the gap is the thing worth closing."*

Same instinct chose Python/FastAPI over Go for the Kubernetes work: *"the aim is to learn Kubernetes,
not to learn Go at the same time."* **Pick one unfamiliar thing per project.**

## Let the app's requirements be the curriculum

KubePlayground's central trick: **the app is deployed into the cluster it observes**, so it cannot
list Pods without RBAC, cannot be reached without a Service and Ingress, cannot survive without
probes and limits.

> *"The app's requirements are the curriculum."*

When a project has a learning goal, **choose a shape where each feature drags a concept in with it.**
It's far more reliable than a syllabus you intend to follow.

## Safeguards ship before the capability

A publicly reachable demo with scale buttons is a live footgun; one that can create arbitrary
workloads is *arbitrary code execution*, and **publicly reachable endpoints that accept workloads get
found by scanners quickly — the usual payload is a crypto miner**, which on a cloud account is a real
bill.

The design that answers it: **the client sends a *choice*, never a spec.** The server renders the
manifest from a fixed template, from a catalogue. Plus a namespaced Role, a ResourceQuota, a
LimitRange, and a CronJob that resets the sandbox — **all of that before the `create` verb exists.**

> *"'Add create now, secure it later' is how a demo becomes an incident."*

And more cheaply: a read-only configuration for the public deployment is the minimum before sharing
a URL at all.

## Name the limitations

> *"Being able to name these matters more than pretending they do not exist."*

Keep a **Known limitations** section and a **Questions worth having an answer ready for** section.
Both are interview preparation disguised as documentation, and writing them surfaces the questions
you *can't* answer yet.

## Related

- `stacks/free-tier-hosting.md` — the platform choices this drives
- `patterns/project-documentation.md` — where these sections live
- `profile/about-jovan.md` — the career context these are aimed at
