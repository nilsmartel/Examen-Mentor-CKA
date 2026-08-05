# CKA Study Progress

> The mentor (Kube) maintains this file. It is the single source of truth for where the learner is.
> Status: `⬜ Not started` · `🟡 In progress` · `✅ Mastered` (only ✅ counts toward completion).
> Overall % is **weighted by exam domain** — see the formula at the bottom.

## 📊 Overall: ~6% complete  *(weighted by exam domain — see formula at bottom)*

`█░░░░░░░░░░░░░░░░░░░` 2 / 27 lessons mastered · the bar tracks **weighted %**, not the raw lesson count

**Strong:** Workloads & Scheduling (2/5) · **Weak / next up:** Cluster Architecture (0/8), Troubleshooting (0/5) — biggest domains, most points on the table

---

## Learner context

- Experience at start: comfortable with pods, deployments, replicasets, services. New to
  cluster admin & troubleshooting depth.
- Background: holds a (now ~3-years-stale) CKAD. First-session diagnostic (2026-08-04) showed
  solid conceptual foundations across rollouts, ConfigMaps/Secrets, Services, and basic RBAC —
  rusty on exact command syntax rather than concepts. Good troubleshooting instincts once
  prompted Socratically. Can move faster through foundation topics than a true beginner; skip
  ground-zero explanations, focus on operational verbs, exact syntax, and exam traps.
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
| 3.1 Deployments, rollouts & rollbacks | ✅ | 2026-08-04 | Diagnostic showed strong prior knowledge; lab (create→update→break→undo→strategy knobs→restart) completed clean. Confidence self-rated 3/5 — re-quiz syntax details. |
| 3.2 ConfigMaps & Secrets | ✅ | 2026-08-05 | Concepts solid immediately (base64≠encryption, env-vs-mount update behavior predicted correctly). Lab parts A-C hands-on (create, env injection, mounted files); part D update-behavior verified by mentor at student's request. Syntax slips en route: invented `--from-label` flag, missing required `name:` on an `env:` entry. Confidence 3/5. |
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
| `rollout undo --to-revision` is absolute, not relative; undo re-numbers the promoted RS to the new highest revision | 3.1 | 2026-08-05 | 2026-08-10 | 5d |
| Editing a live pod doesn't self-heal instantly — it reverts only on next pod replacement, not on edit | 3.1 | 2026-08-05 | 2026-08-06 | 1d |
| env vars from ConfigMap/Secret are captured at pod start; need `kubectl rollout restart` to pick up changes | 3.2 | 2026-08-05 | 2026-08-07 | 2d |
| Mounted ConfigMap/Secret volumes update in place (no restart), env vars don't | 3.2 | 2026-08-05 | 2026-08-07 | 2d |

## ⚠️ Weak spots

_(Mentor: log specific misconceptions or repeated errors here, e.g. "confuses NodePort range",
"forgets to switch context". Target these with deliberate practice.)_

- **Pending vs. image-pull failures (2026-08-04):** initially reached for "check image pull" on a
  `Pending` pod before scheduling. Self-corrected well once prompted Socratically (scheduler must
  place the pod before kubelet can pull anything). Reinforce during 5.1/3.5 — this is a classic
  exam trap when skimming `kubectl get pods` output.
- **Manifest field slips (2026-08-05):** invented a nonexistent `--from-label` flag (conflating CLI
  literals with label metadata) and omitted the required `name:` on an `env:` entry (gave
  `valueFrom` with no name). Both self-corrected with one nudge. Not a conceptual gap — watch for
  more "forgot a required field" slips under exam time pressure; a quick `kubectl explain` habit
  would catch these.

---

## 🗒️ Session log

| Date | Topics covered | Labs done | Outcome / notes |
|------|----------------|-----------|-----------------|
| 2026-08-04 | First-session diagnostic (rollouts, ConfigMaps/Secrets, Services, RBAC, Pending-vs-troubleshooting) + 3.1 Deployments/rollouts | lab-3.1-rollouts (full: update, break, undo, strategy knobs, restart) | 3.1 mastered ✅. Strong foundation confirmed across the board; syntax rusty but concepts solid. Confidence 3/5 — queued for 2-day recall. |

---

## Weighting formula (how the % is computed)

```
overall% = Σ_over_domains ( domain_weight × mastered_in_domain / total_in_domain )
domains:  01=25%/8   02=20%/6   03=15%/5   04=10%/3   05=30%/5
```
Update the **Overall** line and the progress bar every time a lesson's status changes.
The bar has 20 cells; fill `round(overall% / 5)` cells with `█`, rest `░`.
