# CKA Study Progress

> The mentor (Kube) maintains this file — the single source of truth for where the learner is.
> Status: `⬜ Not started` · `🟡 In progress` · `✅ Mastered` (only ✅ counts). Overall % is **weighted by exam domain** (formula at bottom).

## 📊 Overall: ~84% complete

`█████████████████░░░` 22 / 27 lessons mastered · bar tracks **weighted %**, not raw count

**Strong:** Troubleshooting (5/5 ✅) · Storage (3/3 ✅) · **Workloads (5/5 ✅)** · Services & Networking (4/6) · Cluster Architecture (5/8)

> ▶️ **NEXT SESSION:** interleave back to **Services & Networking** — **2.3 Ingress controllers & resources** (weight 20%, well-prepped: 2.2 Services owned, opens the last Svc&Net pair with 2.4 Gateway). **Alts:** 1.6 Helm/Kustomize (guaranteed-earner, hands-on), or 1.8 CRDs & operators (natural follow to 1.7 — CRDs *extend* the API the way CNI/CSI/CRI extend the node; previewed operator read/write-split in 2.2). 5 lessons remain: Domain 01 (1.4 HA, 1.6, 1.8) + Domain 02 (2.3, 2.4).
> ✅ Warm-up watch CLEARED: **single-pod-DNS-blind = EGRESS netpol** — 2026-09-14 he LED with egress (was ingress 3×). Direction now owned; keep in normal rotation.

---

## Learner context

- Holds a ~3-year-stale CKAD. Comfortable with pods/deployments/services; came in rusty on exact command *syntax*, strong on concepts. Good troubleshooting instincts once prompted. Skip ground-zero explanations; focus on operational verbs, exact syntax, and exam traps.
- **Pacing:** adaptive (mentor picks next). **Exam date:** _not set_ (ask; bias pacing to heaviest domains if set).
- **Environment:** minikube + podman on macOS, 2 nodes, **Calico CNI**. See `reference/environment-setup.md`.
- **How he works (honour these — also in agent memory):**
  - Answers **incrementally on purpose** (sends partial replies early because it reads easier). An unanswered sub-question = reading-order, not avoidance. Just re-ask plainly, no commentary. **Never say he "skipped" anything.**
  - **Prefers `kubectl edit` over `patch`** — route him there; don't re-sell patch JSON.
  - **Ask the predict-the-outcome question BEFORE giving commands** — he runs them on sight.
  - **Reasons brilliantly from scenarios, poorly from bare definitions** — frame recall as "pod stuck X, first move?", never "define X".
  - **Grade only fail-SILENT errors** (missing `-n`/`-A`, wrong label/apiGroup, silent wrong answers). Ignore fail-LOUD typos (`log`/`logs`, singular/plural) — the shell + `-h` fix those in ~1s and flagging them annoys him.
  - Under recall pressure he sometimes **invents plausible CLI flags/verbs** rather than saying "unsure". Refuse the guess, split the goal, push `--help`/`kubectl explain` as the reflex.

## ⚠️ Env note (live)

- **Calico CNI token expires periodically on this rebuild.** Symptom: new pods stuck, `describe pod` shows `FailedCreatePodSandBox … calico (add): ClusterInformation: **Unauthorized**` (401 = auth/token, not 403/RBAC). **Fix:** `kubectl rollout restart daemonset/calico-node -n kube-system` regenerates the CNI kubeconfig token on both nodes.
- Namespace has drifted to non-`default` before → `kubectl config get-contexts` if output surprises you.
- metrics-server / storage-provisioner addons enable them if a lab needs `top` / dynamic PVs.
- Interactive `-it` pods are flaky in this relay → use one-shot `kubectl run probe --image=busybox --restart=Never -i --rm -- wget -qO- --timeout=3 http://<svc>`.

---

## Domain 01 — Cluster Architecture, Installation & Config (weight 25%, 8 lessons)

| Lesson | Status | Last | Notes (concise) |
|--------|:------:|:----:|-------|
| 1.1 RBAC | ✅ | 2026-08-07 | Role/RoleBinding (namespaced) vs ClusterRole/Binding (cluster-scope); wrong-apiGroup break/fix; SA + real in-pod token test. Conf 4/5. |
| 1.2 kubeadm install & infra | ✅ | 2026-08-26 | Both trust directions (bootstrap token = cluster→node, may only create a CSR; CA-cert-hash = node→cluster, solves transport not secrecy). kubeconfig = mTLS in YAML. Lab A+C. Self-test 2.5/3. |
| 1.3 Cluster lifecycle & upgrades | ✅ | 2026-08-14 | Drain flags = waivers; cordon/drain/uncordon; upgrade order (kubeadm binary→plan→apply→drain→pkg→restart→uncordon); kubeadm=CP static pods vs pkg mgr=kubelet binary. |
| 1.4 HA control plane | ⬜ | — | |
| 1.5 etcd backup & restore | ✅ | 2026-08-10 | Snapshot save needs 3 certs (live mTLS), restore needs 0 (offline). Restore = restore→new dir + repoint manifest hostPath. Restart cm+scheduler after. |
| 1.6 Helm & Kustomize | ⬜ | — | |
| 1.7 Extension interfaces (CNI/CSI/CRI) | ✅ | 2026-09-16 | Three -I's = jobs k8s refuses to hardcode: CRI=run (crictl NOT docker), CNI=network, CSI=storage. Inspected all 3 live. **Surprise finds, both instructive:** (1) runtime=docker via `cri-dockerd` adapter — untangled minikube driver(podman, builds node) vs in-node runtime(docker, runs pods); (2) `csidrivers` EMPTY yet storage works → minikube-hostpath is a LEGACY provisioner (storage-provisioner pod), not CSI. Asked unprompted whether CSIDriver is a CRD (no — built-in obj; CSINode written by kubelet via node-driver-registrar). CNI=Calico DaemonSet 1/node proven. cgroupDriver-must-match nugget. Self-test 3/3, conf 3/5 (self-assessed the topic as shallow/recognition-level — well-calibrated, not under-confidence). |
| 1.8 CRDs & operators | ⬜ | — | Previewed the operator read/write-split pattern in 2.2. |

## Domain 02 — Services & Networking (weight 20%, 6 lessons)

| Lesson | Status | Last | Notes (concise) |
|--------|:------:|:----:|-------|
| 2.1 Pod connectivity & network model | ✅ | 2026-08-05 | 4 model rules proven hands-on; port-collision break/fix. |
| 2.2 Services & endpoints | ✅ | 2026-08-06 | Service→selector→Ready pods→EndpointSlice→kube-proxy→pod chain; empty-ep vs wrong-targetPort failure modes; headless svc; 3-port trichotomy. Conf 4/5. |
| 2.3 Ingress controllers & resources | ⬜ | — | |
| 2.4 Gateway API | ⬜ | — | |
| 2.5 Network Policies | ✅ | 2026-09-11 | Additive/off-until-touched, deny-by-omission, multi-policy = union. AND/OR dash trap owned. Direction-independence (policyTypes). Default-deny-egress→DNS gotcha. Conf 3/5 (under). |
| 2.6 CoreDNS | ✅ | 2026-09-11 | FQDN `<svc>.<ns>.svc.cluster.local` + search/ndots:5; resolv.conf nameserver = kube-dns ClusterIP as raw number (NAT, no chicken-egg); coredns pods / kube-dns svc naming; break/fix scale coredns. Conf 3/5. |

## Domain 03 — Workloads & Scheduling (weight 15%, 5 lessons — DOMAIN COMPLETE ✅)

| Lesson | Status | Last | Notes (concise) |
|--------|:------:|:----:|-------|
| 3.1 Deployments, rollouts & rollbacks | ✅ | 2026-08-04 | Diagnostic-strong; update→break→undo→strategy→restart clean. |
| 3.2 ConfigMaps & Secrets | ✅ | 2026-08-05 | base64≠encryption; env-vs-mount update behavior predicted correctly. |
| 3.3 Autoscaling (HPA) | ✅ | 2026-09-07 | actual/requested fraction; `<unknown>` fail-silent; `ceil()` one-shot math; Utilization vs AverageValue; scale-up-fast/down-slow. Self-test 3/3. |
| 3.4 Self-healing primitives | ✅ | 2026-09-14 | Three probes proven live: liveness→restart container (RESTARTS++), readiness→endpoint gate (no restart), startup→gates the two then hands off. startup vs liveness = same test, different patience; initialDelay≪boot ⇒ crash-loop of a healthy app, fix=startupProbe. Controller-picker 4/4. StatefulSet needs headless `clusterIP:None`. Self-test 2/2. |
| 3.5 Scheduling (affinity, taints, limits) | ✅ | 2026-09-14 | Two forces: attraction (nodeSelector/affinity, matches NODE labels) vs repulsion (taint) + toleration=PERMIT-not-attract. Combo trap owned: forcing onto a tainted node needs BOTH. 3 effects; NoExecute+tolerationSeconds = the NotReady 5-min evict (connected to 5.1). "Why Pending?"=describe POD not deployment. Fixed a Pending pod live via `edit` (tolerations mutable). Self-test 3/3, conf 4/5. |

## Domain 04 — Storage (weight 10%, 3 lessons — DOMAIN COMPLETE ✅)

| Lesson | Status | Last | Notes (concise) |
|--------|:------:|:----:|-------|
| 4.1 Volumes, access modes, reclaim | ✅ | 2026-09-08 | Container restart = new container (writable layer wiped); emptyDir keyed on pod UID; hostPath node-bound (lived it live); access modes node-scoped; reclaim event-driven, Released = dead end. Self-test 3/3. |
| 4.2 PV & PVC | ✅ | 2026-09-09 | PV/PVC = stable indirection; node-independence is a property of the BACKEND; binding = 3-field shape match; storageClassName 3 states. Self-test 3/3, conf 4/5. |
| 4.3 StorageClasses & dynamic provisioning | ✅ | 2026-09-10 | SC=recipe/provisioner=cook; two live debugs (disabled provisioner addon; RBAC nodes-rule for WFC). volumeBindingMode self-flagged shaky. Conf 4/5. |

## Domain 05 — Troubleshooting (weight 30%, 5 lessons — DOMAIN COMPLETE ✅)

| Lesson | Status | Last | Notes (concise) |
|--------|:------:|:----:|-------|
| 5.1 Cluster & node troubleshooting | ✅ | 2026-08-12 | Debug staircase; conditions polarity; heartbeat-freeze→Unknown; kubelet = systemd service. Two live breaks solo. |
| 5.2 Control-plane components | ✅ | 2026-08-11 | Static-pod model (edit file, kubelet recreates); crictl ps -a on the node; symptom→component map; aftermath vs root cause. |
| 5.3 Resource-usage monitoring | ✅ | 2026-09-03 | metrics chain + APIService aggregation; the two ledgers (requests=reservation / limits=ceiling / top=actual); BestEffort invisible to scheduler. |
| 5.4 Container output & logging | ✅ | 2026-08-27 | Two log streams; `--previous` mechanical (N.log per restart, GC'd, shelf life); exit codes 128+N; Reason vs Exit Code independent; -BackOff = rate-limiter. |
| 5.5 Services & networking troubleshooting | ✅ | 2026-09-11 | Request-path trace; ran wrong-selector / wrong-targetPort / default-deny-netpol breaks cold. endpointslices modern spelling. Conf 4/5. |

---

## 🔁 Spaced-repetition queue

On success push interval (~2d→5d→10d, cap 14d); on a miss reset to ~1d. Retire after 3 clean at cap. Quiz as **scenarios**, not definitions.

| Item / concept | Lesson | Next | Int | Note |
|----------------|:------:|:----:|:---:|------|
| **Three -I interfaces + inspect tool:** CRI=run containers (`crictl version`/`ps -a` on node, NOT docker) · CNI=pod networking (`/etc/cni/net.d/` + DaemonSet 1/node) · CSI=storage (`get csidrivers`, SC `provisioner` field). CRI/CNI node-local (ssh); CSI API-visible | 1.7 | 2026-09-18 | 2d | new; self-test 3/3 |
| **CNI symptom split:** absent CNI ⇒ node **NotReady**; broken-mid-life CNI ⇒ new pods **ContainerCreating** (running pods survive). NetworkPolicy needs a **policy-capable CNI** (Calico/Cilium) — plain **Flannel** gives IPs but silently ignores policies | 1.7/2.5 | 2026-09-18 | 2d | new; got symptom + Flannel trap cold |
| **`csidrivers` EMPTY ≠ no storage.** SC `provisioner` names WHATEVER provisions — a CSI driver (`ebs.csi.aws.com`) OR a legacy/addon provisioner (`k8s.io/minikube-hostpath` = storage-provisioner pod). CSIDriver/CSINode are **built-in objects NOT CRDs**; kubelet writes CSINode via the node-driver-registrar sidecar | 1.7 | 2026-09-18 | 2d | new; nailed "no, legacy provisioners" |
| **minikube two layers:** `--driver=podman` builds the NODE (a podman container on the Mac); the **in-node runtime** runs the pods (here docker via `cri-dockerd` adapter). `crictl version` identifies it. cgroupDriver must MATCH kubelet↔runtime or node breaks | 1.7 | 2026-09-18 | 2d | new; surfaced the docker-not-containerd surprise himself |
| **Taint/toleration polarity:** taint on a NODE repels; toleration on a POD only PERMITS (never attracts). Force a pod onto a tainted node ⇒ need BOTH toleration AND nodeSelector/affinity. Taint key=value matches the pod's TOLERATIONS, never its labels | 3.5 | 2026-09-16 | 2d | new; 3/3 self-test, conf 4/5 |
| **Three taint effects:** NoSchedule (block new) · PreferNoSchedule (soft) · NoExecute (block new + EVICT running non-tolerating). NoExecute + tolerationSeconds = the NotReady node auto-taint that evicts after ~5min (the 5.1 grace) | 3.5 | 2026-09-16 | 2d | new; inferred NoExecute cold |
| **nodeAffinity required vs preferred:** requiredDuringScheduling…=HARD (no match ⇒ Pending) · preferred…=SOFT (schedules anyway). "Why Pending?" ⇒ `describe POD` (scheduler verdict lives there), NOT describe deployment | 3.5 | 2026-09-16 | 2d | new; missed the POD-not-deploy layer once |
| **Three probes & failure action:** liveness→restart container in place · readiness→removed from endpoints, no restart · startup→gates the two, passes once then hands off | 3.4 | 2026-09-16 | 2d | new; crossed liveness↔readiness once |
| **startupProbe = same test, different patience.** Runway = period × failureThreshold > boot; liveness initialDelay ≪ boot ⇒ crash-loop of a HEALTHY app; fix = startupProbe not a bigger delay. Probe fields immutable → edit YAML + recreate | 3.4 | 2026-09-16 | 2d | new; wrote it inverted, self-corrected |
| **Controller picker:** Deployment=N stateless · DaemonSet=1/node (not plain-drainable, `--ignore-daemonsets`) · StatefulSet=stable identity, **needs headless `clusterIP:None`** · Job/CronJob=run-to-completion | 3.4 | 2026-09-19 | 5d | new; 4/4 cold |
| **Single-pod-DNS-blind = that pod's own EGRESS netpol** blocking :53. Cluster-wide DNS down = CoreDNS itself (a **Deployment**, fix w/ kubectl scale/edit). Differentiate CNI-vs-CoreDNS by SCOPE: curl-by-IP works but by-name fails ⇒ CoreDNS; nothing works ⇒ CNI | 2.6/2.5 | 2026-09-19 | 5d | ✅ 09-14 LED with egress (direction cleared) |
| **kube-proxy is NOT in the data path** — per-node daemon that writes NAT rules into the kernel; netfilter is the dataplane; no central hop | 2.6/2.1 | 2026-09-21 | 7d | ✅ clean cold 09-14 |
| **Service DNS FQDN** `<svc>.<ns>.svc.cluster.local` (pods `<ip-dashes>.<ns>.pod...`); short name via `search`+`ndots:5`, first suffix = querying pod's own ns ⇒ cross-ns needs `<svc>.<ns>` | 2.6 | 2026-09-19 | 5d | ✅ clean cold 09-14 |
| **CoreDNS naming:** Deployment/pods = `coredns` (label `k8s-app=kube-dns`); Service = `kube-dns` in kube-system; nameserver = its ClusterIP as raw number, reached by NAT | 2.6 | 2026-09-14 | 3d | |
| **NetworkPolicy** additive/off-until-touched: no policy = allow-all; a policy selecting a pod flips that direction to default-deny; deny-by-omission; multi-policy = UNION (no deny-precedence) | 5.5/2.5 | 2026-09-18 | 7d | |
| **NetworkPolicy AND/OR dash trap:** separate `-` items = OR; podSelector+namespaceSelector under ONE `-` = AND. Count the dashes | 2.5 | 2026-09-18 | 7d | ✅ 2nd cold confirm |
| **NetworkPolicy direction-independence:** `policyTypes` declares which directions opt into deny; an Ingress-only policy leaves egress WIDE OPEN | 2.5 | 2026-09-13 | 2d | got wrong first, one nudge |
| **Default-deny EGRESS silently blocks DNS** — add egress allow to kube-system CoreDNS :53 (UDP *and* TCP) | 2.5 | 2026-09-16 | 5d | |
| **5.5 debug checklist IN ORDER:** endpoints → pods Ready? → targetPort → **DNS** → NetworkPolicy | 5.5 | 2026-09-16 | 5d | watch DNS (drops it) |
| **Empty `get endpoints`** = selector/label mismatch OR pods not Ready; **populated + refused/timeout** = wrong `targetPort`. `get ep` (or endpointslices) is first move | 5.5/2.2 | 2026-09-19 | 5d | ✅ clean cold 09-14 |
| **Three networks fail independently:** node (real ifaces) / pod (CNI's job) / service (ClusterIP = virtual NAT rule). Scheduling ≠ communication. ClusterIP is NAT'd not routed ⇒ **`ping` ClusterIP fails on a healthy Service — test with curl** | 2.1/5.5 | 2026-09-06 | 3d | |
| Service ClusterIP is virtual; kube-proxy translates to a real pod IP. Pod IPs ephemeral → use a Service | 2.1 | 2026-09-11 | 5d | |
| 3 ports: port (client-facing) / targetPort (pod's real listen port) / nodePort (3xxxx). Headless (clusterIP:None) = no VIP, DNS returns pod IPs; +StatefulSet = stable per-pod names | 2.2 | 2026-08-17 | 5d | |
| **`--previous` mechanical:** N.log per restart, kubelet GCs dead containers ⇒ one step back, shelf life ⇒ run it *immediately* on CrashLoop. "previous not found" = never restarted (diagnostic) | 5.4 | 2026-09-08 | 5d | ✅ clean 09-03 |
| **Evidence-location map:** Pending→events (logs impossible) · ImagePullBackOff→events · CrashLoopBackOff→`logs --previous` · OOMKilled→`describe`/Last State. `get events --sort-by=.lastTimestamp` | 5.4 | 2026-08-29 | 2d | |
| **Two log streams:** `journalctl -u kubelet` (node agent, ssh+sudo) vs `kubectl logs` (app stdout, PID 1). App logging to a file inside → `exec` in | 5.4 | 2026-08-29 | 2d | |
| **-BackOff = kubelet retry rate-limiter, never a root cause.** Strip it, ask what failed | 5.4 | 2026-08-29 | 2d | |
| **Exit codes 128+N** (137=SIGKILL, 143=SIGTERM). `Reason:` and `Exit Code:` INDEPENDENT — trust `Reason: OOMKilled` (kubelet reads the cgroup) | 5.4 | 2026-08-29 | 2d | |
| **Stuck-status triage:** `CreateContainerConfigError`=missing CM/Secret · `CrashLoopBackOff`=crash→logs --previous · `Running 0/1`=readiness · `OOMKilled`=over mem · `Terminating`=finalizer/node gone | 5.1 | 2026-09-01 | 5d | |
| **No CNI→NotReady; CNI broken later→ContainerCreating.** Lifecycle→status: pre-schedule=Pending · pulling=ErrImagePull · wiring net=ContainerCreating. `describe pod` **`Node:` empty=scheduler / assigned=kubelet** | 2.1 | 2026-08-31 | 5d | ✅ owned (scenario-framed) |
| **The two ledgers:** requests=RESERVATION (scheduler's only currency, → `describe node` Allocated) / limits=CEILING (kubelet: CPU→throttle, mem→OOMKill) / `top`=actual. "idle node but Pending" = committed≠consumed | 5.3 | 2026-09-12 | 5d | quiz "which command/number", not "explain" |
| **`kubectl run` ⇒ `resources:{}` ⇒ BestEffort ⇒ scheduler charges ZERO ⇒ invisible** (packs more onto it; evicted first). Fix = always set requests | 5.3 | 2026-09-05 | 2d | |
| **Find the hog:** `kubectl top pods -A --sort-by=cpu` — the `-A` IS the question (omitting = fail-silent) | 5.3 | 2026-09-24 | 10d | ✅ fixed the `pod` subresource 09-14 |
| **`kubectl top` needs metrics-server.** Chain kubelet→metrics-server→APIService `metrics.k8s.io`→apiserver proxies. A `1/1 Running` pod ≠ a working API — check `get apiservice v1beta1.metrics.k8s.io`. scoped fail=missing feature; broad fail=broken cluster | 5.3 | 2026-09-20 | 12d | ✅ 3rd clean; near retire |
| **etcd: save needs 3 certs** (--cacert verifies server; --cert/--key = your own matched client pair, key never leaves disk); **restore needs 0** (offline). save=etcdctl, restore=etcdutl | 1.5 | 2026-09-25 | 14d | ✅ clean after 4 misses; retire-track |
| **etcd restore = 2 moves:** restore→new data-dir + repoint the etcd static-pod manifest's etcd-data hostPath. Then restart kube-controller-manager + kube-scheduler (stale watch caches) | 1.5 | 2026-08-17 | 5d | |
| **Static pods** = kubelet runs `/etc/kubernetes/manifests/` files directly (bypasses scheduler/apiserver). Fix = edit the file on the node; `kubectl edit` won't stick | 5.2 | 2026-08-13 | 2d | |
| **API server down** → kubectl refused → ssh, `sudo crictl ps -a` (`-a` = the crashed corpse) + `crictl logs`. **Symptom→component:** apiserver=refused · scheduler=new pods Pending · cm=no self-heal · etcd down presents AS apiserver-down | 5.2 | 2026-08-18 | 5d | |
| **After recovering the control plane, wait 30–60s** — brief NotReady + downstream CrashLoops are aftermath, not root cause. Fix one thing, re-check | 5.2 | 2026-08-18 | 5d | |
| **NotReady node** = kubelet stopped heartbeat → ~40s → conditions flip to **Unknown** (not False). Fix on the NODE via systemd. `inactive`=start; `activating/failed`=read journalctl (bad config), fix `/var/lib/kubelet/config.yaml`, restart | 5.1 | 2026-08-14 | 2d | |
| systemd axes independent: active/inactive (now) vs enabled/disabled (boot); `enable --now`. cordon=no new pods; drain=cordon+evict (`--ignore-daemonsets --delete-emptydir-data`) — each flag a waiver | 5.1/1.3 | 2026-08-16 | 2d | |
| **DaemonSet** = 1 pinned per node; Deployment = N movable. drain skips DS (eviction = pointless recreate loop) | 1.3 | 2026-08-16 | 2d | |
| **kubeadm upgrade order (CP):** kubeadm binary FIRST → plan → apply (rewrites CP static pods, NOT kubelet) → drain → upgrade kubelet/kubectl pkg → daemon-reload+restart → uncordon. Worker step 3 = `upgrade node`. Two mechanisms: kubeadm=CP pods, pkg mgr=kubelet | 1.3 | 2026-09-18 | 10d | ✅ 2nd consecutive |
| Upgrading kubelet = replace binary AND restart (new file on disk ≠ new process); daemon-reload because the pkg edits the unit drop-in | 1.3 | 2026-08-17 | 2d | |
| Two version guardrails: `pkgs.k8s.io` per-minor (magnitude) + `apt-mark hold` (timing; makes OS CVE patching safe). `apt-get update`=index only vs `apt upgrade`=installs (dangerous on a node). Debug versions with `apt-cache policy` | 1.3 | 2026-08-17 | 2d | |
| **Bootstrap token** (cluster trusts node): Secret in kube-system, TTL 24h default (no `expiration` key = never expires), group `system:bootstrappers…` whose entire power = create a CSR. Delete Secret = revoke | 1.2 | 2026-08-28 | 2d | |
| **`--discovery-token-ca-cert-hash`** (node trusts cluster): CA cert is PUBLIC, problem is TRANSPORT not secrecy; fetch CA anon from `cluster-info` CM in kube-public, hash-compare. Like an SSH host-key fingerprint | 1.2 | 2026-08-28 | 2d | derived mechanism, watch motivation |
| **kubeadm init order:** prep every box (containerd+cgroupDriver, pkgs, swapoff, br_netfilter) → init --pod-network-cidr → cp admin.conf → **apply a CNI** → join. kubeadm does NOT install a CNI (missing ⇒ NotReady) | 1.2 | 2026-08-28 | 2d | ⚠️ omitted the CNI step — re-ask |
| **Two kubelet files:** `/etc/kubernetes/kubelet.conf`=IDENTITY (system:node:<name>) vs `/var/lib/kubelet/config.yaml`=BEHAVIOR (staticPodPath, cgroupDriver). Certs at `/etc/kubernetes/pki/`. Don't guess a path — grep the static-pod manifest | 1.2 | 2026-08-28 | 2d | |
| Broken join exam hooks: `kubectl get csr` → `certificate approve`; lost token → `kubeadm token create --print-join-command` | 1.2 | 2026-08-29 | 3d | |
| **kubeconfig = mTLS in YAML** (certificate-authority-data verifies apiserver + client-cert/key prove you). "refused to localhost:8080" = NO config found. ssh'd with no config → `kubectl --kubeconfig=/etc/kubernetes/admin.conf` | 1.2 | 2026-08-28 | 2d | |
| Role/RoleBinding namespaced; cluster-scoped resources REQUIRE ClusterRole+ClusterRoleBinding. Binding sets scope, role sets powers | 1.1 | 2026-08-16 | 5d | |
| `auth can-i` uses `--as` (impersonate), NOT `--user` (kubeconfig entry). can-i never errors (default-deny); the real 403 names group/resource/ns/verb | 1.1 | 2026-08-17 | 5d | |
| Wrong apiGroup silently grants nothing: deployments=`apps`, pods/cm/nodes=core `""`. Imperative `create role` auto-resolves; hand-YAML is where it bites. SA id = `system:serviceaccount:<ns>:<name>`; `edit` is NOT a verb | 1.1 | 2026-08-09 | 2d | |
| **HPA `<unknown>` is FAIL-SILENT** — autoscale succeeds, never scales; TARGETS is the only tell. Causes: (1) no metrics API, (2) no CPU requests. Utilization=`--cpu=50%` (needs requests) vs AverageValue=`--cpu=500m` (needs none) | 3.3 | 2026-09-11 | 3d | said "limit", self-corrected to "requests" |
| **HPA math ONE SHOT:** `ceil(current × metric/target)`, 1→5 immediately. **A reconciling loop always beats a one-shot write** (`kubectl scale` on HPA-managed = overwritten in ~15s) | 3.3 | 2026-09-09 | 2d | quiz as the `kubectl scale` scenario |
| HPA timing: scale-up ~30–60s (3 stacked delays), scale-down waits a **5-min stabilization window**. Slow is normal, not broken | 3.3 | 2026-09-10 | 3d | |
| Container restart = new container from image (writable layer wiped). emptyDir keyed on pod UID → survives restart, dies with pod. hostPath bound to ONE node → survives delete, NOT reschedule ("data vanished overnight" = rescheduled) | 4.1 | 2026-09-14 | 5d | owns hostPath experientially |
| Access modes NODE-scoped: RWO=one node (3 pods same node OK), ROX, RWX (needs NFS-class), RWOP=only pod-scoped one | 4.1 | 2026-09-15 | 5d | |
| Reclaim EVENT-DRIVEN (fires on Bound-PVC delete, not GC). Delete=destroy; Retain=`Released` dead end (won't rebind — payroll-leak). Tell: PVC Pending + PV Released → `edit pv`, remove claimRef | 4.1 | 2026-09-20 | 10d | |
| Pod specs almost entirely IMMUTABLE (image/tolerations mutable) → change a volume = delete+recreate or edit template. PVs are not pods — `edit pv` works | 4.1 | 2026-09-20 | 10d | |
| PV/PVC = stable indirection; **node-independence is a property of the BACKEND** (hostPath-backed PV still node-local, no nodeAffinity). Pod names the PVC never the PV. 3 independent lifecycles | 4.2 | 2026-09-11 | 2d | |
| PVC binding = 3-field shape match (PV cap ≥ request & whole PV consumed; PV modes ⊇; class exact-string). storageClassName: omitted=default/dynamic · `""`=static only · `"name"`=react if class exists else matching key. `describe pvc` events discriminate | 4.2 | 2026-09-11 | 2d | |
| **`volumeBindingMode` (SHAKY, self-flagged):** field on the StorageClass. `Immediate`=provision on PVC create. `WaitForFirstConsumer`=hold until a pod is scheduled, then provision topology-local ⇒ a Pending PVC with no consumer is NORMAL, schedule a pod | 4.3 | 2026-09-15 | 2d | keep short; re-anchor |
| Dynamic provisioning is PLUGGABLE — needs a provisioner pod (CSI driver / minikube addon). WFC makes it need `get nodes` at cluster scope (Immediate doesn't) — a missing RBAC rule is invisible until you switch to WFC | 4.3 | 2026-09-12 | 2d | |

---

## 🗒️ Recent sessions (compact)

| Date | Topic | Outcome |
|------|-------|---------|
| 2026-09-16 | 1.7 Extension interfaces (CNI/CSI/CRI) | ✅ Mastered → Cluster-Arch 5/8, 81→84%. Pure inspection lab, all 3 interfaces ID'd live. Two great surprises he reasoned through: runtime=docker-via-cri-dockerd (untangled podman-driver vs in-node-runtime), and empty `csidrivers` w/ working storage (legacy minikube-hostpath provisioner ≠ CSI). Unprompted depth Q: is CSIDriver a CRD? (no). Warm-up: etcd clean; self-corrected static-pod (first said "edit the controller"); CNI Q3 answered honestly-unsure then owned. Self-test 3/3. |
| 2026-09-14 | 3.5 Scheduling (affinity, taints, limits) | ✅ Mastered → **Workloads DOMAIN COMPLETE (5/5)**, 78→81%. Both forces + combo trap owned; inferred NoExecute cold and connected it to the 5.1 NotReady 5-min evict. Fixed a Pending pod live via `edit` (tolerations mutable). Warm-up: 3/3 cold, incl. leading with EGRESS on the single-pod-DNS item (direction cleared). Self-test 3/3, conf 4/5. |
| 2026-09-14 | 3.4 Self-healing primitives | ✅ Mastered → Workloads 4/5, 75→78%. Probes locked live; controller-picker 4/4. Diagnosed a live Calico 401 CNI break off the events; fixed the intended crash-loop with a startupProbe after catching his own inverted polarity. Self-test 2/2. |
| 2026-09-11 | 2.5 NetworkPolicy, 2.6 CoreDNS, 5.5 Svc-net TS (Calico rebuild) | ✅✅✅ → Troubleshooting DOMAIN COMPLETE (5/5), Svc&Net 4/6, 62→75%. Corrected the kube-proxy-is-a-hop misconception. |
| 2026-09-10 | 4.3 StorageClasses & dynamic provisioning | ✅ → Storage COMPLETE (3/3), 59→62%. Two exam-grade live debugs (disabled addon; RBAC nodes-rule for WFC). |
| 2026-09-09 | 4.2 PV & PVC | ✅ 55→59%. Node-independence = backend property; storageClassName 3 states. Self-test 3/3. |
| 2026-09-08 | 4.1 Volumes/access modes/reclaim | ✅ 52→55% — every domain now on the board. Lived the hostPath node-trap. Gave the "stop saying skipped" + fail-loud/silent feedback here. |
| 2026-09-07 | 3.3 HPA | ✅ 49→52%. Found `--cpu-percent` deprecated himself; corrected incremental-scaling + scale-beats-HPA misconceptions. |
| 2026-09-03 | 5.3 Resource monitoring | ✅ 43→49% (crossed halfway). The two-ledgers capstone; BestEffort-invisible. |

_(Older sessions 2026-08-04 → 08-27 established: 1.1 RBAC, 1.2 kubeadm, 1.3 lifecycle, 1.5 etcd, 2.1/2.2 networking, 3.1/3.2 workloads, 5.1/5.2/5.4 troubleshooting. Details compacted into the domain tables + recall queue above.)_

---

## Weighting formula

```
overall% = Σ ( domain_weight × mastered_in_domain / total_in_domain )
domains:  01=25%/8   02=20%/6   03=15%/5   04=10%/3   05=30%/5
```
Current: 01=25%×5/8=15.6 · 02=20%×4/6=13.3 · 03=15%×5/5=15.0 · 04=10%×3/3=10.0 · 05=30%×5/5=30.0 → **~84.0% ≈ 84%**.
Bar = 20 cells, `round(% / 5)` filled. Update Overall + bar whenever a status changes.
