# KubePlayground

**Repo:** https://github.com/Jovan253/KubePlayground · **State as of:** 2026-09-05 (last push)

An interactive web app that visualises a **live Kubernetes cluster** — Pods, Deployments, Services,
Nodes — and lets you scale and inspect workloads from the browser.

Two jobs at once: a **learning vehicle for the KCNA exam** and an **interactive portfolio piece**.

---

## The design trick

**The app is deployed into the cluster it observes.** That's the whole teaching mechanism — the app
cannot list Pods without a ServiceAccount and RBAC, cannot be reached without a Service and Ingress,
cannot survive without probes and limits.

> *"The app's requirements are the curriculum."*

## Stack

- **Backend:** Python 3.12 + FastAPI + the official `kubernetes` client (chosen over Go's `client-go`
  so he wasn't learning Go and Kubernetes simultaneously)
- **Frontend:** plain HTML/CSS/JS, served by FastAPI
- **Packaging:** Helm chart in `chart/`, with `values-local.yaml` / `values-eks.yaml`
- **Infra:** Terraform in `terraform/` — GitHub OIDC, ECR, VPC, EKS
- **CI:** GitHub Actions in `.github/`, deliberately manual-trigger
- **Clusters:** local `docker-desktop`, plus EKS raised on demand

## ⚠️ Working mode — this repo is the exception

**He writes the Kubernetes manifests himself.** Explain the concept, name the file, describe what it
must contain conceptually — then **stop and wait**. Don't pre-emptively create `.yaml`.

Application code may be written for him; that's plumbing. Reviewing, debugging and quizzing are
welcome. Full detail in `profile/working-agreement.md`.

## Its own docs

- **`ROADMAP.md` is the source of truth** for what's done and next — read it before assuming
  anything. It's also a detailed learning log, milestone by milestone (M0–M12).
- `CLAUDE.md` — working mode, environment, context-safety rule
- `README.md`, `kcna-cheatsheet.html`

## Where it got to

- **Reached:** M1–M4 (objects by hand, containerised, RBAC, Service, Ingress), M4b (three sites
  behind one controller), M5/M5b (frontend, ownership tree, YAML pane, kubectl transcript), M6
  (probes and measured resource limits), M8 (Helm chart), M9 steps 1–5 (OIDC, ECR, VPC, EKS, access)
- **Publicly reachable on EKS, 2026-09-01**, via a `LoadBalancer` Service
- **Known open items:** the deploy pipeline has not run end to end (the chart was upgraded by hand);
  scale buttons are live to anyone with the URL; no HTTPS; `terraform destroy` for the node group
  when not demoing — and note the load balancer belongs to the Helm release, so destroying only the
  cluster can strand it
- **Planned but not started:** M7 observability, M10 safe workload catalogue, M11 one ingress
  controller + three sites on EKS, M12 the demo GIF

## Lessons it contributed

Most of the Kubernetes, Helm and AWS content in this store:

- `stacks/kubernetes.md` — probes, RBAC, resource limits and CFS throttling, Ingress, image caching
- `stacks/helm.md` — the immutable-selector trap, label helpers, `nindent`, release ownership
- `stacks/aws-terraform-and-oidc.md` — the immutable OIDC subject claim, EKS's two permission
  systems, EKS cost shaping the architecture
- `patterns/portfolio-project-criteria.md` — "let the requirements be the curriculum", safeguards
  before capability
