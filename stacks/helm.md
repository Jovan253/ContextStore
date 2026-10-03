# Helm

From packaging **KubePlayground** (Sep 2026), verified with helm v4.2.4.

## Why a chart appeared at all

Not for its own sake. The forcing question was: **the same manifests must point at a locally-built
tag on the dev cluster and at an ECR path on EKS.** Raw YAML cannot do both. If you don't have that
problem yet, raw manifests are fine and simpler.

Structure that worked:

```
chart/
├─ Chart.yaml
├─ values.yaml          defaults
├─ values-local.yaml    per-cluster override
└─ values-eks.yaml      per-cluster override
```

CI overrides the tag per deploy with `--set image.tag=$GITHUB_SHA`, so the running image always
traces back to a commit.

## The four things that bit

- **Two label helpers, not one.** `selectorLabels` — a stable subset — goes in
  `spec.selector.matchLabels` and the pod template. The full `labels` set, which carries the chart
  version, goes **only** on `metadata`.

  **A Deployment's selector is immutable**, so a version label inside it makes every single
  `helm upgrade` fail. This is the most common self-inflicted Helm wound.

- **No `metadata.namespace` in any template.** Helm supplies it from `--namespace`. Hardcoding it
  makes the chart installable in exactly one place.

- **ClusterRole and ClusterRoleBinding are cluster-scoped**, so the release namespace does *not*
  keep two installs apart — they will collide. Prefix their names with `.Release.Name`.

- **Helm will not adopt objects that `kubectl apply` created.** The existing raw resources have to
  be deleted from the cluster before the first `helm install`, or it fails on ownership. That is
  the practical face of "a release is tracked, an apply is not". Note the *files* can stay in the
  repo; it's the live objects that must go.

## `helm lint` is not enough

`helm lint` checks chart **structure**. Only `helm template` actually renders and parses the
output, which is what catches indentation errors. Run both.

**`nindent` is the trap.** `include` returns a **multi-line** string, so without `nindent` only the
first line lands at the right column and the rest is garbage. `{{-` strips the preceding whitespace
and `nindent N` puts back exactly one newline plus N spaces per line — **they are a matched pair**,
use them together.

**`toYaml` only works where the value's shape ALREADY IS the target YAML.** This is why it suits
`resources` (values mirror the Kubernetes schema exactly) but **not** `ports`, where the values are
an abstraction (`container.port`) over a list with different key names. If your values are a
friendlier shape than the API's, `toYaml` will emit nonsense.

## Cloud interaction worth knowing

A `type: LoadBalancer` Service in a chart means **the cloud load balancer belongs to the Helm
release, not to Terraform.** `helm uninstall` removes it; `terraform destroy` targeting only the
cluster can **strand** it and keep billing. Order your teardown accordingly.

## Related

- `stacks/kubernetes.md` — the objects being templated, and why the selector is immutable
- `stacks/aws-terraform-and-oidc.md` — the CI identity that installs the chart, and why it ends up
  needing more permission than you'd like
