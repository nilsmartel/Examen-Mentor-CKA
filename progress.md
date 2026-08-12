# CKA Study Progress

> The mentor (Kube) maintains this file. It is the single source of truth for where the learner is.
> Status: `⬜ Not started` · `🟡 In progress` · `✅ Mastered` (only ✅ counts toward completion).
> Overall % is **weighted by exam domain** — see the formula at the bottom.

## 📊 Overall: ~31% complete  *(weighted by exam domain — see formula at bottom)*

`██████░░░░░░░░░░░░░░` 8 / 27 lessons mastered · the bar tracks **weighted %**, not the raw lesson count

> ▶️ **NEXT SESSION — start here:** 5.1 node troubleshooting **done ✅** — Troubleshooting now **2/5** (30% domain).
> Broke the kubelet TWO ways, both diagnosed cold: (B) `systemctl stop` → node NotReady, all conditions **Unknown**
> (frozen heartbeat), fixed with `enable --now`. (E) corrupt `/var/lib/kubelet/config.yaml` → crash-loop, read the
> `yaml: line 57` parse error in journalctl, fixed the file + restart. Learned active-vs-enabled, cordon vs drain.
> **Best next step: interleave into Domain 01 — `1.3` cluster lifecycle & upgrades** (kubeadm upgrade, node
> drain/cordon which we just previewed — heavy 25% earner, natural bridge). *Alternatives:* `5.4` container logs/exit
> codes (easy Troubleshooting win, guaranteed points) or `5.3` metrics-server/`top`. Warm-up recall: new 5.1 items
> (Unknown-conditions = frozen heartbeat; inactive-vs-crashloop; parse-error≠missing-file) + still-due static-path/CNI.

**Strong:** Troubleshooting (2/5: control-plane + node/kubelet repair — both break/fix cold), Cluster Architecture (2/8: RBAC + etcd), Workloads (2/5), Services & Networking (2/6) · **Weak / next up:** Troubleshooting still 3 to go (30% weight, most points — keep drilling: 5.4/5.3/5.5), Cluster Architecture 6 to go (kubeadm/upgrades/HA — start 1.3 next)

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
| 1.5 etcd backup & restore | ✅ | 2026-08-10 | **Mastered.** Opened with a full certs/mTLS primer (built it Socratically: key pair → sign-with-private/verify-with-public → the trust gap → CA-signed cert binds pubkey↔identity → mTLS two-way → 3 etcdctl flags map onto the handshake). Learner derived the sign/verify trick, the "stolen photocopy needs the private key" impersonation point, and the ca.crt/crt-public vs key-secret split himself. Lab full A–D hands-on: snapshot save (verified w/ etcdutl status), created after-backup post-snapshot, restore into new dir, repointed manifest hostPath (nailed the "change ONLY the etcd-data hostPath, not --data-dir/mountPath" trap), watched after-backup vanish. **Standout insights:** asked what restore *does* (snap.db=archive vs restored/=bootable data-dir); asked about orphaned containers → led to the reconcile-loop explanation (kubelet GCs orphaned pods; restore rewinds desired state, controllers drag actual→desired). Grasped save-needs-3-certs (live mTLS) vs restore-needs-none (offline file op). **Minikube wrinkle handled:** host has no etcdctl + distroless image → drove via `kubectl exec etcd-minikube`, writing only to mounted paths. Confidence high. ~16%→~19%. |
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
| 5.1 Cluster & node troubleshooting | ✅ | 2026-08-12 | **Mastered** (2nd Troubleshooting win). Taught the debug staircase (get nodes/pods → describe → logs → ssh+journalctl) + the polarity trap (Pressure conditions healthy=False, Ready healthy=True) + heartbeat-freeze (kubelet dies → LastHeartbeatTime freezes → conditions flip to **Unknown** not False after ~40s). Lab full: **Part C** mentor stopped kubelet → node NotReady, diagnosed cold (pods fine, node NotReady, all conditions Unknown → kubelet not responding), read `journalctl`, fixed with `systemctl enable --now`. Bumped into active-vs-enabled-vs-preset (taught: active/inactive=running now [start/stop]; enabled/disabled=on boot [enable/disable]; preset=distro default). **Part E** mentor corrupted `/var/lib/kubelet/config.yaml` (appended bad YAML) → kubelet crash-loops → **initially misread the error as "file not found"** (corrected: file present, contents malformed — read the FULL line `yaml: line 57: did not find expected node content`), also floated "restart kubeadm" (corrected: kubeadm is a one-shot CLI, not a service, doesn't repopulate). Fixed the file → self-healed via `Restart=always` before he even typed restart (taught: don't rely on it; explicit restart, daemon-reload only for unit-file edits). Closed with cordon (SchedulingDisabled/spec.unschedulable, no NEW pods) vs drain (cordon+evict, --ignore-daemonsets/--delete-emptydir-data) vs uncordon. Slips: `sysctl` vs `systemctl` typo; parse-vs-missing misread. Confidence 3/5 (modest — drove both breaks solo). ~25%→~31%. |
| 5.2 Control-plane components | ✅ | 2026-08-11 | **Mastered** (first Troubleshooting win — the 30% domain opens). Taught the static-pod model Socratically: killed the "scheduler controls the nodes" misconception with the bootstrap chicken-and-egg (kubelet runs `/etc/kubernetes/manifests/` FILES directly, bypassing scheduler+apiserver) → derived exam-trap #1 (can't `kubectl edit` a static pod, edit the file, kubelet recreates). Lab A–C full hands-on: **Part B** mystery scheduler break (bad image tag → ErrImagePull → reproduced "new pods stuck Pending" → fixed by reading the correct tag from another manifest, verified testpod schedules onto minikube). **Part C** the classic apiserver break (I seeded bad `--etcd-servers` port 9999) → kubectl died → diagnosed ENTIRELY on the node: `crictl ps -a` (taught why `-a`: evidence is in the crashed corpse) + `crictl logs` → read the fatal `context deadline exceeded` / etcd `connection refused` → found correct port 2379 by reading etcd.yaml (didn't guess) → fixed manifest → kubelet recreated → cluster back. **Standout:** independently spotted storage-provisioner CrashLoopBackOff and, once he saw it self-heal, correctly classified it as *aftermath* not *root cause* — the exact cause-vs-collateral discrimination the exam rewards. Also learned the post-recovery "wait 30–60s, node briefly NotReady is just heartbeat re-posting" patience lesson. Slip: ran `crictl` without sudo → go-panic logs (resolved: needs sudo). Confidence 3/5 (slightly under — mostly solo). ~19%→~25%. |
| 5.3 Resource-usage monitoring | ⬜ | — | |
| 5.4 Container output & logging | ⬜ | — | |
| 5.5 Services & networking troubleshooting | ⬜ | — | |

---

## 🔁 Spaced-repetition queue

Items to re-quiz, with next review date. Add an item when a lesson hits ✅.
On success push the interval out (~2d → ~5d → ~10d); on a miss reset to ~1d.

| Item / concept | Lesson | Last reviewed | Next review | Interval |
|----------------|:------:|:-------------:|:-----------:|:--------:|
| `rollout undo --to-revision` is absolute, not relative; undo re-numbers the promoted RS to the new highest revision | 3.1 | 2026-08-10 | 2026-08-20 | 10d |
| Editing a live pod doesn't self-heal instantly — RS reconciles pod *count* not spec; reverts only on pod *replacement* | 3.1 | 2026-08-08 | 2026-08-13 | 5d |
| _(retired 2026-08-11 — owned)_ env vars captured at pod start (need `rollout restart`) vs mounted volumes update in place. Learner asked to stop quizzing; nailed 4×. | 3.2 | 2026-08-11 | — | retired |
| No CNI → nodes `NotReady`; CNI broken later → pods `ContainerCreating` (plugin present but can't wire the sandbox/give an IP; `describe pod` shows the CNI sandbox error). Fix = apply/repair CNI manifest, not restart apiserver/kubelet | 2.1 | 2026-08-12 | 2026-08-14 | 2d (soft miss: knew NotReady half, blanked on ContainerCreating) |
| Service ClusterIP is virtual (no real interface); kube-proxy translates it to a pod IP. Pod IP is a real endpoint | 2.1 | 2026-08-06 | 2026-08-11 | 5d |
| Pod IPs are ephemeral (new IP on recreate) → never hardcode; use a Service for a stable VIP/DNS | 2.1 | 2026-08-06 | 2026-08-11 | 5d |
| Service debug chain: Service(selector)→Ready pods→EndpointSlice→kube-proxy→pod. `kubectl get ep` is first move | 2.2 | 2026-08-08 | 2026-08-13 | 5d |
| Empty endpoints = selector mismatch OR pods not Ready; populated endpoints but "refused" = wrong targetPort | 2.2 | 2026-08-08 | 2026-08-13 | 5d |
| 3 ports: port (svc, client-facing) / targetPort (pod's real listen port, must match app) / nodePort (external 3xxxx) | 2.2 | 2026-08-12 | 2026-08-17 | 5d |
| Headless Service (clusterIP:None) = no VIP; DNS returns pod IPs; +StatefulSet gives stable per-pod names (web-0.web…) | 2.2 | 2026-08-10 | 2026-08-15 | 5d |
| etcd `snapshot save` needs 3 certs (live mTLS: cacert verifies etcd, cert+key = your client identity); `snapshot restore` needs ZERO certs (offline file op, no endpoint). Restore uses `etcdutl`, save uses `etcdctl` | 1.5 | 2026-08-11 | 2026-08-16 | 5d |
| etcd restore = 2 moves: (1) `etcdutl snapshot restore snap.db --data-dir=NEW` (offline), (2) repoint the etcd static-pod manifest's **etcd-data hostPath** to NEW (leave `--data-dir` flag + mountPath). kubelet restarts etcd | 1.5 | 2026-08-12 | 2026-08-17 | 5d |
| AFTER an etcd restore, restart kube-controller-manager + kube-scheduler — their watch caches hold future resourceVersions the rewound etcd no longer has → stalls (e.g. namespace stuck Terminating). apiserver self-restarts. Restart a static pod by moving its manifest out of /etc/kubernetes/manifests and back | 1.5 | 2026-08-12 | 2026-08-17 | 5d (✅ nailed both components + the watch-cache reason) |
| Locate static-pod manifests: standard = `/etc/kubernetes/manifests/` (**/etc** not /var — it's config); if unsure derive it via `grep staticPodPath /var/lib/kubelet/config.yaml`. Cert paths + endpoint are read straight from `etcd.yaml` | 1.5 | 2026-08-12 | 2026-08-13 | 1d (❌ said /var/kubernetes; also missed the confirm cmd) |
| Role/RoleBinding are namespaced; cluster-scoped resources (nodes, PVs, namespaces) REQUIRE ClusterRole+ClusterRoleBinding — a Role can't grant them. The binding sets the scope, the role sets the powers | 1.1 | 2026-08-11 | 2026-08-16 | 5d |
| Static pods = kubelet runs manifests in `/etc/kubernetes/manifests/` FILES directly (bypasses scheduler+apiserver, so control plane can bootstrap). FIX = edit the file on the node → kubelet auto-recreates; `kubectl edit`/`apply` won't stick | 5.2 | 2026-08-11 | 2026-08-13 | 2d |
| API server DOWN → `kubectl` = connection refused → ssh to node. `sudo crictl ps -a` (`-a` shows the CRASHED container — evidence is in the corpse; needs sudo) + `sudo crictl logs <id>` for the fatal error | 5.2 | 2026-08-11 | 2026-08-13 | 2d |
| Symptom→component map: apiserver down = kubectl refused; scheduler down = new pods stuck **Pending** (nodes healthy, existing pods fine); controller-manager down = no self-heal/endpoints; etcd down = apiserver won't start | 5.2 | 2026-08-11 | 2026-08-13 | 2d |
| After recovering the control plane, wait 30–60s: a brief node `NotReady` + downstream CrashLoops (e.g. storage-provisioner) are **aftermath** (lost apiserver connection), not root cause. Fix the ONE real thing; collateral self-heals | 5.2 | 2026-08-11 | 2026-08-13 | 2d |
| `auth can-i` uses `--as` (impersonate); `--user` picks a kubeconfig entry (→ "does not exist"). can-i never errors (default-deny); the real request gives a 403 naming group/resource/ns/verb | 1.1 | 2026-08-12 | 2026-08-17 | 5d |
| Wrong apiGroup silently grants nothing: deployments=`apps`, pods/configmaps/nodes=core `""`. Imperative `create role --resource=` auto-resolves the group; hand-written YAML is where the trap bites | 1.1 | 2026-08-07 | 2026-08-09 | 2d |
| SA identity in bindings/impersonation = `system:serviceaccount:<ns>:<name>`; SAs are real namespaced objects (fully-qualified), Users/Groups are just trusted strings (no object). `edit` is NOT a verb → use `update`/`patch` | 1.1 | 2026-08-07 | 2026-08-09 | 2d |
| NotReady node = kubelet stopped posting heartbeat → after ~40s conditions flip to **Unknown** (not False — node-controller hears nothing). `LastHeartbeatTime` freezes. Fix lives on the NODE (systemd), not the API. kubelet is a **systemd service** → logs via `journalctl -u kubelet`, not crictl/kubectl | 5.1 | 2026-08-12 | 2026-08-14 | 2d |
| Two kubelet failure modes, told apart by `systemctl status`: **`inactive (dead)`** = just stopped → `systemctl start`. **`activating (auto-restart)`/`failed`** = crash-looping on bad config → READ `journalctl` (e.g. `yaml: line N` = parse error, file PRESENT not missing), fix `/var/lib/kubelet/config.yaml`, `restart`. `Restart=always` self-heals once fixed but don't wait; `daemon-reload` only after editing a unit FILE | 5.1 | 2026-08-12 | 2026-08-14 | 2d |
| systemd axes are independent: **active/inactive** (running now; `start`/`stop`) vs **enabled/disabled** (auto-start on boot; `enable`/`disable`); `preset` = distro default. `enable --now` = start + enable in one. cordon = `SchedulingDisabled` (spec.unschedulable, no NEW pods, existing stay) vs drain = cordon + EVICT (`--ignore-daemonsets --delete-emptydir-data`); `uncordon` to restore | 5.1 | 2026-08-12 | 2026-08-14 | 2d |

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
- **Certificates / PKI gap (2026-08-08 → RESOLVED 2026-08-10):** did the full primer to open the session.
  Learner built the model himself Socratically — sign-with-private/verify-with-public, the trust gap,
  CA-signed cert binds pubkey↔identity, mTLS two-way, and the "stolen photocopy can't sign without the
  private key" impersonation point. Cleanly stated ca.crt/crt = public-shareable vs key = secret-revoke.
  One residual slip to watch: under quiz he mislabeled the 3 etcdctl flags as "etcd's key + apiserver's
  pubkey" instead of CA cert + *client's own* cert + *client's own* key (the lab's server-cert-reused-as-
  client-cert shortcut probably confused him). Recall item covers it. Foundation now solid for 1.2/5.2.
- **`crictl` needs `sudo` (2026-08-11):** ran `crictl` un-privileged during 5.2 → got a Go panic/connection stack trace, briefly thought crictl was broken. It's just root-only (runtime socket). On the node, always `sudo crictl …`. Low stakes, one-time nudge — flag if it recurs when the API server is down and crictl is the *only* tool.
- **CNI naming (recurring):** said "CDI" for CNI again (3rd time, from 2.1). Diagnosis/reasoning is
  correct every time — purely the acronym. Had him say "CNI" back. Low stakes but worth a nudge if it recurs.
- **Error-message misread: "not found" vs "malformed" (2026-08-12, 5.1):** on the corrupt-kubelet-config
  break he read `failed to load kubelet config file` as "no file found" and proposed regenerating via kubeadm,
  when the file was present and only its *contents* were bad (`yaml: line 57`). Corrected by making him paste the
  FULL journalctl line. **Exam-relevant pattern:** slow down and read the whole error — "load failed / parse error"
  (fix the contents) is a different bug from "not found" (regenerate). Watch for this under time pressure.
- **`sysctl` vs `systemctl` (2026-08-12):** typo'd `sysctl` (kernel tunables) for `systemctl` (service manager).
  One-time; flagged for exam-speed muscle memory. Also `/var/kubernetes` vs correct `/etc/kubernetes` on recall.

---

## 🗒️ Session log

| Date | Topics covered | Labs done | Outcome / notes |
|------|----------------|-----------|-----------------|
| 2026-08-04 | First-session diagnostic (rollouts, ConfigMaps/Secrets, Services, RBAC, Pending-vs-troubleshooting) + 3.1 Deployments/rollouts | lab-3.1-rollouts (full: update, break, undo, strategy knobs, restart) | 3.1 mastered ✅. Strong foundation confirmed across the board; syntax rusty but concepts solid. Confidence 3/5 — queued for 2-day recall. |
| 2026-08-05 | Recall warm-up (3.1: rollback revision, live-pod edit) + 3.2 ConfigMaps & Secrets | lab-3.2-config-secrets (A-C hands-on; D update-behavior mentor-verified) | 3.2 mastered ✅. Base64≠encryption and env-vs-mount update behavior both predicted correctly pre-lab. Minor manifest-field slips, self-corrected. Confidence 3/5 — queued for 2-day recall. |
| 2026-08-05 | Ad-hoc recall (3.1 rollout cmds, 3.2 env-var staleness) + 2.1 Pod connectivity & network model (CIDR/NAT/flat-network explained from scratch) | lab-2.1-pod-connectivity (A cross-pod curl by IP, B ephemeral IPs, C shared netns; break-it/fix-it: two-nginx port-80 collision) | 2.1 mastered ✅. Productive struggle — initially doubted pods have IPs, disproved own hypothesis via `get pods -o wide`. Needed CNI taught directly (didn't know the component). Wobbles: "CDI" for CNI; unaware Service IP is virtual. Confidence 3/5. |
| 2026-08-06 | Recall (3.1 live-pod edit, 2.1 ClusterIP-virtual + ephemeral IPs — all solid) + 2.2 Services & Endpoints. Foundational Q first: how many pods share a node/IPs → network namespaces & per-namespace port space (tied back to 2.1 shared-netns lab). | lab-2.2-services A–D: ClusterIP + wget-by-DNS, break selector→empty ep→fix, NodePort + 3-port trichotomy, targetPort→8080 mismatch→refused→fix | 2.2 mastered ✅. Very curious/self-driving session — asked about api-resources short names, EndpointSlice naming, headless-vs-hardcoded-IP, read/write replica split, and independently derived the label+operator failover pattern (1.8 preview). Brief StatefulSet intro. Confidence 4/5 (highest yet). |
| 2026-08-07 | Recall (3.2 mounted-vs-env update, 2.1 CNI-broken→NotReady — both correct) + finished 1.1 RBAC (namespaced-vs-cluster probe, then full lab). | lab-1.1-rbac A–D: User/Role/RoleBinding + `can-i --as`, wrong-apiGroup break/fix (read the 403 naming `apps`), SA + real in-pod token test, ClusterRole-reused-per-ns via RoleBinding | 1.1 mastered ✅ (first Domain 01 win). Curiosity tangents: *why* apiGroups exist (groups/versioning/extensibility), full verb list. Slips: `--user` vs `--as` on can-i; "edit" not a verb; "CDI"→CNI again. SA-vs-User confusion now resolved. Confidence 4/5. ~13%→~16%. |
| 2026-08-08 | Recall (3× all correct: Service debug chain / empty-vs-refused endpoints / live-pod edit reverts) + **1.5 etcd concepts** (etcd=cluster DB, snapshot save/restore, mTLS+3 cert flags, restore-to-new-dir→repoint manifest, `minikube ssh` rationale). Short session — learner had to leave right before the lab. | none (stopped before lab Part A) | 1.5 **in progress 🟡**, not mastered (no lab, no self-test → no % change, stays ~16%). Great boundary insight: etcd snapshot ≠ volume data. mTLS was new → **requested a certificates/identity primer to open next session**. Confidence n/a. |
| 2026-08-10 | Recall (3 warm-ups: 3-port trichotomy [soft miss — swapped port↔targetPort, corrected], headless Service [ok], rollout undo --to-revision [ok]) → **Certificates/mTLS primer** (built Socratically) → **1.5 etcd lab A–D**. | lab-1.5-etcd-backup-restore FULL: snapshot save (3 certs) + etcdutl status verify, created after-backup post-snapshot, etcdutl restore→new data dir, repointed manifest hostPath, confirmed after-backup vanished. **Bonus real troubleshooting:** stale mirror-pod status (Pending vs crictl-Running) + **namespace stuck Terminating** → diagnosed via cm logs (watch at rV 118936 > restored rev 117930) → fixed by restarting controller-manager + scheduler static pods (manifest-move technique). | **1.5 mastered ✅** (2nd Domain 01 win). Deep, curiosity-driven session: derived the mTLS model himself, asked snap.db(archive) vs restored/(bootable data-dir), and the orphaned-container Q → reconcile-loop teaching. Learned the restore-aftermath control-plane restart (guaranteed exam earner). Cert/PKI gap resolved. ~16%→~19%. |
| 2026-08-11 | Recall (3× all correct: RBAC cluster-scoped-needs-ClusterRole, ConfigMap env-vs-mount [retired — owns it], etcd save-3-certs/restore-zero + etcdctl-vs-etcdutl bonus) → **5.2 Control-plane / static-pod troubleshooting** (taught Socratically, killed the "scheduler controls nodes" idea via the bootstrap chicken-and-egg). | lab-5.2-control-plane A–C FULL: **B** mystery scheduler break (bad image tag→ErrImagePull→reproduced pods-stuck-Pending→fixed by reading correct tag from another manifest→verified testpod schedules). **C** classic apiserver break (mentor-seeded bad `--etcd-servers` port 9999)→kubectl dead→diagnosed ON NODE with `crictl ps -a`+`crictl logs` (read fatal `context deadline exceeded`/etcd refused)→found port 2379 from etcd.yaml→fixed→recovered. Unscripted: spotted storage-provisioner CrashLoop, classified it as aftermath not root cause when it self-healed. | **5.2 mastered ✅** — **first Troubleshooting win (30% domain opens)**. Mostly solo, strong. Learned: static-pod fix = edit file/kubelet recreates (no apply); `crictl -a` + why; symptom→component map; post-recovery patience (brief NotReady = heartbeat). Slip: `crictl` without sudo → go-panic (needs sudo). Confidence 3/5. ~19%→~25%. |
| 2026-08-12 | **Quick 20-min recall round** (3× all correct: etcd restore 2-moves, NodePort 3-port trichotomy, `--as` vs `--user` + edit-not-a-verb — all pushed to 5d). Then **5.1 primer seed**: NotReady-node scenario. Learner didn't know node health model yet (guessed kube-controller-manager). Taught the kubelet-heartbeat/Lease model: kubelet posts heartbeat → node-controller (in kcm) is a *reader* that flips NotReady after silence; fix = `describe node` then `systemctl status kubelet` / `journalctl -u kubelet` (kubelet is a **systemd service**, not a static pod). | none (recall + concept only) | No status change (~25%). Primed 5.1 for a full break/fix next session. Spine landed: NotReady → kubelet dead/misconfigured → read its systemd logs. |
| 2026-08-12 (cont.) | Warm-up recall (1½/3: etcd-restore-restart ✅ strong; CNI NotReady/ContainerCreating ½ — blanked ContainerCreating; static-pod path ❌ said /var). Then **5.1 node troubleshooting FULL lab** — debug staircase, conditions polarity trap, heartbeat-freeze→Unknown. | lab-5.1 C+E+stretch: **C** kubelet stopped (mentor) → NotReady, diagnosed cold, fixed `enable --now`; learned active/enabled/preset. **E** corrupt `/var/lib/kubelet/config.yaml` → crash-loop, read `yaml: line 57`, fixed file (self-healed via Restart=always); learned parse≠missing, daemon-reload rule. cordon vs drain vs uncordon. | **5.1 mastered ✅** (2nd Troubleshooting win, domain now 2/5). Drove both breaks solo — strong. Slips: read "load failed" as "file not found" + proposed kubeadm-restart (corrected); `sysctl`/`systemctl` typo. Confidence 3/5 (modest). ~25%→~31%. |

---

## Weighting formula (how the % is computed)

```
overall% = Σ_over_domains ( domain_weight × mastered_in_domain / total_in_domain )
domains:  01=25%/8   02=20%/6   03=15%/5   04=10%/3   05=30%/5
```
Update the **Overall** line and the progress bar every time a lesson's status changes.
The bar has 20 cells; fill `round(overall% / 5)` cells with `█`, rest `░`.
