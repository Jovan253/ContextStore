# AWS, Terraform, GitHub OIDC and EKS

From **KubePlayground**'s M9 (Sep 2026). Context: Jovan writes Terraform daily but had not used
AWS before this — so the unfamiliar part is the *resources*, not the language. Explain what a
resource is and why AWS needs it; don't explain HCL.

---

## GitHub OIDC — the gotcha that cost a full debugging round

Use OIDC federation, never long-lived access keys in repo secrets. It is also the single most
interview-relevant detail in a pipeline.

**GitHub issues IMMUTABLE subject claims.** The trust policy was written the way every tutorial
shows it:

```
repo:Jovan253/KubePlayground:ref:refs/heads/main
```

and the token actually said:

```
repo:Jovan253@54801590/KubePlayground@1346312309:ref:refs/heads/main
```

**Every name carries its numeric ID.** `StringEquals` is exact, so it never matched — and the
error, *"Not authorized to perform sts:AssumeRoleWithWebIdentity"*, says nothing whatsoever about
why.

**This is a security feature, not a bug.** Names are mutable: delete a repo or rename an account,
someone else claims the name, and a policy matching plain names would then trust *their* workflows.
Numeric IDs are never reused. So the fix is to **match the immutable form** — never to disable it.

### The technique worth keeping

**Don't guess at claim mismatches.** A workflow step can request its own OIDC token and print
selected claims. Print `sub` and nothing sensitive — **the token itself is a bearer credential and
must never reach a log.**

Things ruled out first, in order, all of them wrong: provider exists with the right audience; trust
policy contents; **canonical repo casing** (`StringEquals` is case-sensitive and a git remote
preserves whatever you typed, not GitHub's canonical name — a real candidate); default branch; IAM
propagation timing.

## EKS access needs TWO permission systems, and you need both

- **`eks:DescribeCluster` in AWS IAM** — this is what `aws eks update-kubeconfig` calls to fetch the
  endpoint and CA. Grant only this and you still cannot touch a single Kubernetes object.
- **An access entry + a policy association** — registers the IAM role as a principal the cluster
  recognises, then binds it to permissions. **The entry alone grants nothing**, exactly like a
  ServiceAccount with no RoleBinding.

**Authentication is AWS's job; authorisation stays Kubernetes'.** So a 403 from `kubectl` on EKS
means asking *which of the two* refused you. That question saves a lot of time.

### Least-privilege debt, and the honest production split

If your chart creates a ClusterRole, Kubernetes' **escalation prevention** means a namespace-scoped
CI identity *physically cannot* install it — you cannot create a role granting permissions you do
not already hold. The shortcut is to grant CI `AmazonEKSClusterAdminPolicy` and move on, which is
what happened here, deliberately and knowingly.

The fix when it matters:

- `rbac.create: false` in values
- a human admin provisions the ClusterRole/ClusterRoleBinding once
- CI drops to `AmazonEKSAdminPolicy` scoped to the namespace

**Cluster-scoped by a human, namespaced by CI** is the normal production split.

## Cost — this shapes the architecture, not just the bill

**EKS costs $0.10/hr for the control plane alone (~$73/month) whether or not anything runs on it**,
plus nodes, load balancer and NAT.

So for a personal/portfolio project, build the environment to be **raised and destroyed on demand**,
with `destroy` as a manually-triggerable workflow. *"The pipeline stands the whole environment up
from nothing"* is a better story than a cluster idling at $100/mo anyway.

Other cost facts learned:

- **A NAT gateway costs real money.** Keeping nodes in public subnets avoids it — a legitimate
  trade for a sandbox, not for production.
- A `LoadBalancer` Service on EKS is **~$16–20/month** on top of the cluster.
- Three `LoadBalancer` Services = three load balancers ≈ $50/mo to demonstrate *worse* architecture
  than one ingress controller doing path-based routing. See `stacks/kubernetes.md`.

## ECR

- The registry becomes necessary at exactly the point the node stops being your laptop:
  `imagePullPolicy: IfNotPresent` with a locally-built tag stops working.
- Add a **lifecycle policy** when you create the repository, not later.
- **The node pulls using `AmazonEC2ContainerRegistryReadOnly` on its instance role** — no
  `imagePullSecret` and **no Kubernetes ServiceAccount involved in the pull at all**. Worth
  internalising: image pulls are a kubelet concern, not a workload-identity concern.

## Operational notes

- **`aws eks update-kubeconfig` makes EKS the CURRENT context.** Rename the context and switch back
  to your sandbox immediately, so an unqualified `kubectl delete` hits the playground and not AWS.
- **An `EXTERNAL-IP` that exists is not an endpoint that answers.** After the load balancer hostname
  appeared it took **~75 seconds** before it served traffic — its health checks have to pass first.
- A LoadBalancer Service gives you plain **HTTP** on an AWS hostname. HTTPS needs the AWS Load
  Balancer Controller plus ACM, which is meaningfully more setup — budget for it as its own piece of
  work, not a flag.
- The load balancer belongs to the **Helm release**, so `terraform destroy` on the cluster alone can
  strand it. See `stacks/helm.md`.
- On EKS you will see workloads that never existed locally: `aws-node` (the VPC CNI) and
  `kube-proxy` as DaemonSets.

## Ordering that worked

Incrementally, one AWS concept at a time, each landing separately:

1. **Identity** — GitHub OIDC provider + the IAM role Actions assumes
2. **Registry** — ECR repository + lifecycle policy
3. **Network** — VPC, subnets, routing (why EKS wants multiple AZs; why NAT costs money)
4. **Cluster** — EKS control plane + a small managed node group
5. **Access** — the two permission systems above
6. **Then** deploy by hand once, to prove the chain, before automating it

Step 6 is the one people skip. Proving the chain manually first means a pipeline failure is a
pipeline problem, not an everything problem.

## Related

- `stacks/kubernetes.md` · `stacks/helm.md` · `patterns/debugging-playbook.md`
- `projects/kubeplayground.md` — current state, including what hasn't run end to end yet
