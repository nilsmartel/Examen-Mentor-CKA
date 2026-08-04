# CKA Curriculum Map

> This is the mentor's master index. It defines every lesson, its exam-domain weight,
> and the order dependencies. The `progress.md` weighting formula is derived from this file.
> **Kubernetes version targeted: v1.35** (current exam). Curriculum reflects the Feb-2025 update.

## Exam facts (tell the student when relevant, don't quiz on trivia)

- **Format:** Online, remote-proctored, **performance-based** (hands-on in a real cluster terminal). No multiple choice.
- **Duration:** 2 hours.
- **Passing score:** 66%.
- **Attempts:** 1 free retake included.
- **Allowed during exam:** exactly ONE browser tab open to the official docs —
  `kubernetes.io/docs`, `kubernetes.io/blog`, plus `helm.sh/docs`. Nothing else.
- **Clusters:** Multiple clusters; each task tells you which `kubectl config use-context <name>` to switch to. **Always switch context first** — a huge source of lost points.
- **You may `ssh` into nodes** and use `sudo`. Some tasks require it (kubelet, static pods, etcd).

## Domains, weights, and lessons

Overall completion is a **weighted** number (see `progress.md`). Within a domain every lesson
counts equally; each domain contributes its exam weight.

| # | Domain | Weight | Lessons |
|---|--------|:------:|:-------:|
| 01 | Cluster Architecture, Installation & Configuration | 25% | 8 |
| 02 | Services & Networking | 20% | 6 |
| 03 | Workloads & Scheduling | 15% | 5 |
| 04 | Storage | 10% | 3 |
| 05 | Troubleshooting | 30% | 5 |
| | **Total** | **100%** | **27** |

### 01 — Cluster Architecture, Installation & Configuration (25%)
- `1.1` RBAC — roles, bindings, service accounts, `kubectl auth can-i`
- `1.2` kubeadm install & infra prerequisites (container runtime, CNI, join tokens)  *(concept-heavy on minikube)*
- `1.3` Cluster lifecycle — kubeadm upgrades, node drain/cordon, kubelet versioning
- `1.4` Highly-available control plane — stacked vs external etcd, load balancer  *(concept-heavy on minikube)*
- `1.5` etcd backup & restore  *(fully practicable via `minikube ssh`)*
- `1.6` Helm & Kustomize for cluster components
- `1.7` Extension interfaces — CNI, CSI, CRI (what each is, how to inspect)
- `1.8` CRDs, installing & configuring operators

### 02 — Services & Networking (20%)
- `2.1` Pod connectivity & the Kubernetes networking model
- `2.2` Services & endpoints — ClusterIP, NodePort, LoadBalancer, EndpointSlices
- `2.3` Ingress controllers & Ingress resources
- `2.4` Gateway API — GatewayClass, Gateway, HTTPRoute (new in 2025)
- `2.5` Network Policies
- `2.6` CoreDNS & cluster DNS

### 03 — Workloads & Scheduling (15%)
- `3.1` Deployments, rolling updates & rollbacks
- `3.2` ConfigMaps & Secrets for app configuration
- `3.3` Workload autoscaling — HPA (+ metrics-server)
- `3.4` Self-healing primitives — probes, ReplicaSets, DaemonSets, StatefulSets
- `3.5` Scheduling — requests/limits, node selectors/affinity, taints & tolerations

### 04 — Storage (10%)
- `4.1` Volumes — types, access modes, reclaim policies
- `4.2` PersistentVolumes & PersistentVolumeClaims
- `4.3` StorageClasses & dynamic provisioning

### 05 — Troubleshooting (30%)
- `5.1` Cluster & node troubleshooting (kubelet, NotReady nodes)
- `5.2` Control-plane component troubleshooting (static pods, API server, etcd)
- `5.3` Monitoring resource usage (metrics-server, `kubectl top`)
- `5.4` Container output streams & logging (`kubectl logs`, exit codes, events)
- `5.5` Services & networking troubleshooting

## Suggested adaptive order (not a fixed schedule)

Because pacing is adaptive, the mentor chooses the *next* lesson using these rules:
1. **Foundation first:** confirm 2.1, 2.2, 3.1, 3.2 quickly (student already knows the basics) before advanced topics.
2. **Weight-biased:** favor Troubleshooting (30%) and Cluster Architecture (25%) once foundations are set — they carry the most points.
3. **Dependency-aware:** 5.2 needs 1.5 (etcd); 2.3/2.4 need 2.2; 3.3 needs metrics-server (5.3); 4.2 needs 4.1.
4. **Interleave:** don't do all of one domain back-to-back — alternate domains to strengthen discrimination between concepts (evidence-based; see `reference/teaching-methodology.md`).

A reasonable default path:
`3.1 → 3.2 → 2.1 → 2.2 → 5.4 → 1.1 → 5.3 → 3.3 → 5.1 → 4.1 → 4.2 → 1.5 → 5.2 → 2.5 → 2.6 → 2.3 → 2.4 → 4.3 → 3.4 → 3.5 → 1.3 → 1.6 → 1.7 → 1.8 → 1.2 → 1.4 → 5.5`
