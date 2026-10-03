# About Jovan

Calibration file. Read this to pitch explanations at the right level instead of guessing.

*Facts below are dated. Where something is likely to have moved on since, it says so.*

---

## Role and experience

**Cloud/DevOps engineer.** ~2 years of professional experience as of **August 2026**; this is his
first job after university.

- **Strong on:** CI/CD pipelines, **Terraform**, **Azure**. Reads and writes IaC daily.
- **Comfortable with:** declarative config, YAML, cloud IAM concepts, container basics.
- **Was new to (Aug 2026):** **Kubernetes** — had watched course material but done essentially no
  hands-on work before starting KubePlayground. Working toward the **KCNA** exam.
- **Was new to (Sep 2026):** **AWS** specifically. Writes Terraform daily, so *the language is not
  the gap — the resources are.* Explain what an AWS resource is and why AWS needs it; don't explain
  HCL.
- Also an **AI-103** certification in late Sep 2026.

**Do not assume Kubernetes fluency** (object model, kubectl muscle memory, cluster internals) if the
conversation is dated before ~late 2026 — but do check `projects/kubeplayground.md`, since a lot of
ground was covered. **Do assume** IaC, pipeline and cloud-IAM fluency.

## Languages and stacks he actually uses

Evidenced across his repos:

| Area | Where |
|---|---|
| Terraform / HCL | KubePlayground, azure-infrastructure-example |
| Python + FastAPI | KubePlayground backend, TrackSplit API, AI-Foundations |
| React + TypeScript | Mythos, TrackSplit web, PutMeOn |
| React + JavaScript | portfolio, sudoku-solver, TripChecker, GrooveTracker |
| Luau / Roblox | RobloxAppDigSell |
| Go | MyFinOps |
| C# / .NET | MovieRanker |
| C++ | Electronics-1 |

So: **polyglot, but the centre of gravity is infrastructure plus Python/TypeScript glue.**

## How he picks projects

- **Deliberately targets gaps rather than day-job skills.** Chose AWS + GitHub Actions over
  Azure + GitLab specifically because the latter are already day-job skills.
- **One unfamiliar thing at a time.** Chose FastAPI over Go for the Kubernetes work so he wasn't
  learning two things at once.
- **Portfolio and learning in one artefact** — see `patterns/portfolio-project-criteria.md`.
- **Cost-sensitive to the point of it being architectural.** Free tier or nothing for personal
  projects.

## What he's good at that's worth leaning on

From the record, these ideas were *his*, not suggested to him:

- Visualising the Deployment → ReplicaSet → Pod ownership chain as a tree
- Running two extra sites behind the same ingress controller to show what a controller is *for*
- The hub-and-spoke map layout, radial zones, wooden signs, moving platforms
- The Collection Journal, and splitting discovery from claiming the reward
- Pickaxe skins via cases with a spinning reveal, rarer skins with particle effects

**He has good product instincts and will push back on design.** Present reasoning and a
recommendation; expect him to redirect it, often improving it.

## Communication

- He'll tell you plainly when something's off: *"it feels quite dense"*, *"slightly too hard"*,
  *"stop guessing"*. Take it literally, and act on the specific words.
- He confirms things by **actually running them** and reports back. Give him something testable and
  say what to look for.
- He asks for assessments (*"is the pacing right?"*) and wants **computed answers**, not opinions.

## Related

- `profile/working-agreement.md` — how he wants an agent to behave
- `profile/dev-machine-windows.md` — the machine
