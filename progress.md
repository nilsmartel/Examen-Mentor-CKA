# CKA Study Progress

> The mentor (Kube) maintains this file. It is the single source of truth for where the learner is.
> Status: `⬜ Not started` · `🟡 In progress` · `✅ Mastered` (only ✅ counts toward completion).
> Overall % is **weighted by exam domain** — see the formula at the bottom.

## 📊 Overall: ~16% complete  *(weighted by exam domain — see formula at bottom)*

`███░░░░░░░░░░░░░░░░░` 5 / 27 lessons mastered · the bar tracks **weighted %**, not the raw lesson count

**Strong:** Workloads & Scheduling (2/5), Services & Networking building (2/6: pod model + Services), RBAC now solid (1/8) · **Weak / next up:** Cluster Architecture (1/8 — 7 to go, incl. etcd/upgrades/HA), Troubleshooting (0/5) — biggest domains, most points on the table

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
| 1.1 RBAC | ✅ | 2026-08-07 | **Mastered.** Finished the namespaced-vs-cluster probe (nailed: cluster-wide pod read = ClusterRole+ClusterRoleBinding; RoleBinding→ClusterRole limits to one ns; two RoleBindings for dev+prod vs ClusterRoleBinding for all). Lab 1.1 A–D all hands-on: User/Role/RoleBinding, ServiceAccount + real in-pod token test (exec'd kubectl in a pod running as the SA — saw the 403 for `default`), ClusterRole-reused-per-ns. Break/fix: wrong-apiGroup (edited role apiGroups→`[""]`, watched deployments read silently break, read the 403 that names `apps`, fixed). Great curiosity: asked why apiGroups exist (taught groups/versioning/extensibility) + verb list. Confidence 4/5. |
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
| 2.1 Pod connectivity & network model | ✅ | 2026-08-05 | Proved all 4 model rules hands-on (distinct pod IPs, cross-pod curl by IP, ephemeral IP on recreate, shared-netns via localhost). Break-it/fix-it: two-nginx port-80 collision → "Address already in use" → split into separate pods. Self-tests passed; wobbles: said "CDI" for CNI (recapped), and didn't know Service ClusterIP is virtual/kube-proxy-translated (clarified). Confidence 3/5. |
| 2.2 Services & endpoints | ✅ | 2026-08-06 | Derived the whole model well. Nailed the chain (Service→selector→Ready pods→EndpointSlice→kube-proxy→pod) and both failure modes: empty ep = selector/readiness; populated-but-refused = wrong targetPort (verified hands-on via patch to 8080). Labs A–D done incl. two break/fix. Strong tangents self-driven: netns/port-sharing recap, EndpointSlice naming (<svc>-rand, find by label), port/targetPort/nodePort trichotomy, headless (clusterIP:None) vs hardcoded-IP, and derived the operator/label read-write-split pattern unprompted (previews 1.8). Brief StatefulSet intro given. Confidence 4/5. |
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
| Editing a live pod doesn't self-heal instantly — RS reconciles pod *count* not spec; reverts only on pod *replacement* | 3.1 | 2026-08-06 | 2026-08-08 | 2d |
| env vars from ConfigMap/Secret are captured at pod start; need `kubectl rollout restart` to pick up changes | 3.2 | 2026-08-05 | 2026-08-10 | 5d |
| Mounted ConfigMap/Secret volumes update in place (no restart), env vars don't | 3.2 | 2026-08-07 | 2026-08-12 | 5d |
| No CNI → nodes `NotReady`; CNI broken later → pods `ContainerCreating`. Fix = apply/repair CNI manifest, not restart apiserver/kubelet | 2.1 | 2026-08-07 | 2026-08-12 | 5d |
| Service ClusterIP is virtual (no real interface); kube-proxy translates it to a pod IP. Pod IP is a real endpoint | 2.1 | 2026-08-06 | 2026-08-11 | 5d |
| Pod IPs are ephemeral (new IP on recreate) → never hardcode; use a Service for a stable VIP/DNS | 2.1 | 2026-08-06 | 2026-08-11 | 5d |
| Service debug chain: Service(selector)→Ready pods→EndpointSlice→kube-proxy→pod. `kubectl get ep` is first move | 2.2 | 2026-08-06 | 2026-08-08 | 2d |
| Empty endpoints = selector mismatch OR pods not Ready; populated endpoints but "refused" = wrong targetPort | 2.2 | 2026-08-06 | 2026-08-08 | 2d |
| 3 ports: port (svc, client-facing) / targetPort (pod's real listen port, must match app) / nodePort (external 3xxxx) | 2.2 | 2026-08-06 | 2026-08-08 | 2d |
| Headless Service (clusterIP:None) = no VIP; DNS returns pod IPs; +StatefulSet gives stable per-pod names (web-0.web…) | 2.2 | 2026-08-06 | 2026-08-08 | 2d |
| Role/RoleBinding are namespaced; cluster-scoped resources (nodes, PVs, namespaces) REQUIRE ClusterRole+ClusterRoleBinding — a Role can't grant them. The binding sets the scope, the role sets the powers | 1.1 | 2026-08-07 | 2026-08-09 | 2d |
| `auth can-i` uses `--as` (impersonate); `--user` picks a kubeconfig entry (→ "does not exist"). can-i never errors (default-deny); the real request gives a 403 naming group/resource/ns/verb | 1.1 | 2026-08-07 | 2026-08-09 | 2d |
| Wrong apiGroup silently grants nothing: deployments=`apps`, pods/configmaps/nodes=core `""`. Imperative `create role --resource=` auto-resolves the group; hand-written YAML is where the trap bites | 1.1 | 2026-08-07 | 2026-08-09 | 2d |
| SA identity in bindings/impersonation = `system:serviceaccount:<ns>:<name>`; SAs are real namespaced objects (fully-qualified), Users/Groups are just trusted strings (no object). `edit` is NOT a verb → use `update`/`patch` | 1.1 | 2026-08-07 | 2026-08-09 | 2d |

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
- **RBAC: SA vs User conflation (2026-08-06→RESOLVED 2026-08-07):** previously thought humans auth *as*
  SAs. In the 1.1 lab he articulated the split cleanly (Users = trusted strings/no object; SAs = real
  namespaced objects, `system:serviceaccount:ns:name`) and verified it via a real in-pod token test.
  Considered resolved — keep the recall item live.
- **`--as` vs `--user` on `auth can-i` (2026-08-07):** used `--user=jane` → "jane does not exist".
  `--user` selects a *kubeconfig* entry; impersonation is `--as`. Quick slip, corrected; watch under time
  pressure. Also flagged: `edit` is not an RBAC verb (→ `update`/`patch`).
- **CNI naming (recurring):** said "CDI" for CNI again (3rd time, from 2.1). Diagnosis/reasoning is
  correct every time — purely the acronym. Had him say "CNI" back. Low stakes but worth a nudge if it recurs.

---

## 🗒️ Session log

| Date | Topics covered | Labs done | Outcome / notes |
|------|----------------|-----------|-----------------|
| 2026-08-04 | First-session diagnostic (rollouts, ConfigMaps/Secrets, Services, RBAC, Pending-vs-troubleshooting) + 3.1 Deployments/rollouts | lab-3.1-rollouts (full: update, break, undo, strategy knobs, restart) | 3.1 mastered ✅. Strong foundation confirmed across the board; syntax rusty but concepts solid. Confidence 3/5 — queued for 2-day recall. |
| 2026-08-05 | Recall warm-up (3.1: rollback revision, live-pod edit) + 3.2 ConfigMaps & Secrets | lab-3.2-config-secrets (A-C hands-on; D update-behavior mentor-verified) | 3.2 mastered ✅. Base64≠encryption and env-vs-mount update behavior both predicted correctly pre-lab. Minor manifest-field slips, self-corrected. Confidence 3/5 — queued for 2-day recall. |
| 2026-08-05 | Ad-hoc recall (3.1 rollout cmds, 3.2 env-var staleness) + 2.1 Pod connectivity & network model (CIDR/NAT/flat-network explained from scratch) | lab-2.1-pod-connectivity (A cross-pod curl by IP, B ephemeral IPs, C shared netns; break-it/fix-it: two-nginx port-80 collision) | 2.1 mastered ✅. Productive struggle — initially doubted pods have IPs, disproved own hypothesis via `get pods -o wide`. Needed CNI taught directly (didn't know the component). Wobbles: "CDI" for CNI; unaware Service IP is virtual. Confidence 3/5. |
| 2026-08-06 | Recall (3.1 live-pod edit, 2.1 ClusterIP-virtual + ephemeral IPs — all solid) + 2.2 Services & Endpoints. Foundational Q first: how many pods share a node/IPs → network namespaces & per-namespace port space (tied back to 2.1 shared-netns lab). | lab-2.2-services A–D: ClusterIP + wget-by-DNS, break selector→empty ep→fix, NodePort + 3-port trichotomy, targetPort→8080 mismatch→refused→fix | 2.2 mastered ✅. Very curious/self-driving session — asked about api-resources short names, EndpointSlice naming, headless-vs-hardcoded-IP, read/write replica split, and independently derived the label+operator failover pattern (1.8 preview). Brief StatefulSet intro. Confidence 4/5 (highest yet). |
| 2026-08-07 | Recall (3.2 mounted-vs-env update, 2.1 CNI-broken→NotReady — both correct) + finished 1.1 RBAC (namespaced-vs-cluster probe, then full lab). | lab-1.1-rbac A–D: User/Role/RoleBinding + `can-i --as`, wrong-apiGroup break/fix (read the 403 naming `apps`), SA + real in-pod token test, ClusterRole-reused-per-ns via RoleBinding | 1.1 mastered ✅ (first Domain 01 win). Curiosity tangents: *why* apiGroups exist (groups/versioning/extensibility), full verb list. Slips: `--user` vs `--as` on can-i; "edit" not a verb; "CDI"→CNI again. SA-vs-User confusion now resolved. Confidence 4/5. ~13%→~16%. |

---

## Weighting formula (how the % is computed)

```
overall% = Σ_over_domains ( domain_weight × mastered_in_domain / total_in_domain )
domains:  01=25%/8   02=20%/6   03=15%/5   04=10%/3   05=30%/5
```
Update the **Overall** line and the progress bar every time a lesson's status changes.
The bar has 20 cells; fill `round(overall% / 5)` cells with `█`, rest `░`.
