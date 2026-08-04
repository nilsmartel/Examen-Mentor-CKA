# CKA Study Progress

> The mentor (Kube) maintains this file. It is the single source of truth for where the learner is.
> Status: `⬜ Not started` · `🟡 In progress` · `✅ Mastered` (only ✅ counts toward completion).
> Overall % is **weighted by exam domain** — see the formula at the bottom.

## 📊 Overall: ~0% complete  *(weighted by exam domain — see formula at bottom)*

`░░░░░░░░░░░░░░░░░░░░` 0 / 27 lessons mastered · the bar tracks **weighted %**, not the raw lesson count

**Strong:** _(none yet)_ · **Weak / next up:** _(run the first-session diagnostic)_

---

## Learner context

- Experience at start: comfortable with pods, deployments, replicasets, services. New to
  cluster admin & troubleshooting depth.
- Pacing: **adaptive** (mentor chooses next topic).
- Target exam date: _not set_ (ask the learner; if set, note it here and bias pacing).
- Environment: minikube + podman on macOS (see `reference/environment-setup.md`).

---

## Domain 01 — Cluster Architecture, Installation & Config (weight 25%, 8 lessons)

| Lesson | Status | Last touched | Notes |
|--------|:------:|:------------:|-------|
| 1.1 RBAC | ⬜ | — | |
| 1.2 kubeadm install & infra | ⬜ | — | |
| 1.3 Cluster lifecycle & upgrades | ⬜ | — | |
| 1.4 HA control plane | ⬜ | — | |
| 1.5 etcd backup & restore | ⬜ | — | |
| 1.6 Helm & Kustomize | ⬜ | — | |
| 1.7 Extension interfaces (CNI/CSI/CRI) | ⬜ | — | |
| 1.8 CRDs & operators | ⬜ | — | |

## Domain 02 — Services & Networking (weight 20%, 6 lessons)

| Lesson | Status | Last touched | Notes |
|--------|:------:|:------------:|-------|
| 2.1 Pod connectivity & network model | ⬜ | — | |
| 2.2 Services & endpoints | ⬜ | — | |
| 2.3 Ingress controllers & resources | ⬜ | — | |
| 2.4 Gateway API | ⬜ | — | |
| 2.5 Network Policies | ⬜ | — | |
| 2.6 CoreDNS | ⬜ | — | |

## Domain 03 — Workloads & Scheduling (weight 15%, 5 lessons)

| Lesson | Status | Last touched | Notes |
|--------|:------:|:------------:|-------|
| 3.1 Deployments, rollouts & rollbacks | ⬜ | — | |
| 3.2 ConfigMaps & Secrets | ⬜ | — | |
| 3.3 Autoscaling (HPA) | ⬜ | — | |
| 3.4 Self-healing primitives | ⬜ | — | |
| 3.5 Scheduling (affinity, taints, limits) | ⬜ | — | |

## Domain 04 — Storage (weight 10%, 3 lessons)

| Lesson | Status | Last touched | Notes |
|--------|:------:|:------------:|-------|
| 4.1 Volumes, access modes, reclaim | ⬜ | — | |
| 4.2 PV & PVC | ⬜ | — | |
| 4.3 StorageClasses & dynamic provisioning | ⬜ | — | |

## Domain 05 — Troubleshooting (weight 30%, 5 lessons)

| Lesson | Status | Last touched | Notes |
|--------|:------:|:------------:|-------|
| 5.1 Cluster & node troubleshooting | ⬜ | — | |
| 5.2 Control-plane components | ⬜ | — | |
| 5.3 Resource-usage monitoring | ⬜ | — | |
| 5.4 Container output & logging | ⬜ | — | |
| 5.5 Services & networking troubleshooting | ⬜ | — | |

---

## 🔁 Spaced-repetition queue

Items to re-quiz, with next review date. Add an item when a lesson hits ✅.
On success push the interval out (~2d → ~5d → ~10d); on a miss reset to ~1d.

| Item / concept | Lesson | Last reviewed | Next review | Interval |
|----------------|:------:|:-------------:|:-----------:|:--------:|
| _(none yet)_ | | | | |

## ⚠️ Weak spots

_(Mentor: log specific misconceptions or repeated errors here, e.g. "confuses NodePort range",
"forgets to switch context". Target these with deliberate practice.)_

---

## 🗒️ Session log

| Date | Topics covered | Labs done | Outcome / notes |
|------|----------------|-----------|-----------------|
| _(none yet)_ | | | |

---

## Weighting formula (how the % is computed)

```
overall% = Σ_over_domains ( domain_weight × mastered_in_domain / total_in_domain )
domains:  01=25%/8   02=20%/6   03=15%/5   04=10%/3   05=30%/5
```
Update the **Overall** line and the progress bar every time a lesson's status changes.
The bar has 20 cells; fill `round(overall% / 5)` cells with `█`, rest `░`.
