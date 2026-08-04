# Recall Bank (spaced-repetition question bank)

The mentor draws from this for the **warm-up recall** at the start of each session, two ways:
(1) for **scheduled reviews**, ask the `✅` items that are *due* in `progress.md`'s recall queue;
(2) **ad hoc**, you may also pull questions for a recently-taught `🟡` lesson to reinforce it. Only
`✅` items get scheduled in the queue. Ask 2–3 per session; on a miss, re-teach and reset that item to
~1 day (cap intervals at ~14d; retire an item after 3 straight successes at the cap — see CLAUDE.md §6).

> Format: `Q` then `A:` (the model answer, for the mentor — don't read it out; use it to grade).
> Keep it active — prefer "do X" / "what command" over "define X".

## Domain 01 — Cluster Architecture
- **1.1** You need jane to read pods only in `web`. Which two objects, and the verify command?
  A: Role + RoleBinding in `web`; `kubectl auth can-i get pods -n web --as=jane`.
- **1.1** Grant one ClusterRole across three namespaces — how? A: three RoleBindings (one per ns) → the ClusterRole.
- **1.1** Which API group are deployments in? A: `apps` (not core `""`).
- **1.2** After `kubeadm init`, all nodes NotReady — why? A: no CNI installed yet.
- **1.2** Lost the worker join command — regenerate it? A: `kubeadm token create --print-join-command`.
- **1.3** Worker upgrade command vs control-plane? A: worker `kubeadm upgrade node`; CP `kubeadm upgrade apply vX.Y.Z`.
- **1.3** `drain` errors on DaemonSet pods — flag? A: `--ignore-daemonsets`.
- **1.4** etcd members to tolerate 1 failure? A: 3 (quorum 2). Use odd numbers.
- **1.4** Stacked vs external etcd? A: stacked = etcd on the control-plane nodes; external = dedicated etcd nodes.
- **1.5** The three cert flags every etcdctl command needs, and where to find their paths?
  A: `--cacert/--cert/--key`; from `/etc/kubernetes/manifests/etcd.yaml`.
- **1.5** After `snapshot restore --data-dir=/var/lib/etcd-restored`, what must change? A: the etcd static-pod manifest's hostPath → the restored dir.
- **1.6** Preview a Helm chart without installing? A: `helm template` (or `--dry-run`).
- **1.6** Apply a Kustomize overlay? A: `kubectl apply -k <dir>`.
- **1.7** Which interface missing keeps nodes NotReady? A: CNI.
- **1.8** What does a CRD alone do? A: adds a type/stores objects; needs a controller (operator) to act.

## Domain 02 — Services & Networking
- **2.1** Pod A on node1 → pod B on node2 by IP — works? A: yes, no NAT (CNI provides the flat network).
- **2.2** Service does nothing — first command? A: `kubectl get endpoints <svc>` (empty ⇒ selector/label/readiness).
- **2.2** `port` vs `targetPort`? A: `port` = Service's client port; `targetPort` = pod's port.
- **2.2** Type superset order? A: LoadBalancer ⊃ NodePort ⊃ ClusterIP.
- **2.3** Applied an Ingress, nothing happens — cause? A: no ingress controller running.
- **2.3** Mandatory field in each Ingress path? A: `pathType`.
- **2.4** Gateway API's three resources + owners? A: GatewayClass (vendor), Gateway (infra), HTTPRoute (dev).
- **2.4** How does an HTTPRoute attach to a Gateway? A: `parentRefs`.
- **2.5** Pod selected by a policy, no ingress rules listed — what's allowed? A: nothing (default-deny that direction).
- **2.5** Egress default-deny broke DNS — why? A: port 53 to CoreDNS now blocked; allow it.
- **2.6** FQDN of Service `api` in ns `shop`? A: `api.shop.svc.cluster.local`.
- **2.6** DNS down cluster-wide — first two checks? A: CoreDNS pods (kube-system) and the `kube-dns` Service.

## Domain 03 — Workloads & Scheduling
- **3.1** Update image + watch — two commands? A: `kubectl set image ...`; `kubectl rollout status deploy/x`.
- **3.1** Roll back to revision 1? A: `kubectl rollout undo deploy/x --to-revision=1`.
- **3.2** Updated a ConfigMap; env-var consumers unchanged — why/fix? A: env read at start; `rollout restart` (or mount as files).
- **3.2** Is a Secret encrypted by default? A: no — base64; needs encryption-at-rest + RBAC.
- **3.3** HPA `TARGETS: <unknown>` — two causes? A: no metrics-server; or pods lack CPU requests.
- **3.3** Create HPA for deploy web, 50% CPU, 1–10? A: `kubectl autoscale deploy web --cpu-percent=50 --min=1 --max=10`.
- **3.4** Readiness probe fails — pod restarted? A: no; removed from endpoints. (Liveness restarts.)
- **3.4** One pod per node for an agent — controller? A: DaemonSet.
- **3.5** Pod Pending after tainting its only node — fix? A: add matching toleration or remove taint (`...:NoSchedule-`).
- **3.5** Memory-limit exceeded vs CPU-limit? A: memory → OOMKilled (137); CPU → throttled.

## Domain 04 — Storage
- **4.1** emptyDir pod deleted — data? A: gone (pod-scoped).
- **4.1** RWO vs RWX? A: RWO one node RW; RWX many nodes RW (needs shared storage).
- **4.1** Reclaim Retain vs Delete when PVC deleted? A: Retain keeps PV/data (manual); Delete removes PV + backend.
- **4.2** PVC Pending — three things that must match a PV? A: capacity, access modes, storageClassName.
- **4.2** Pod references PV or PVC? A: the PVC.
- **4.3** What creates the PV in dynamic provisioning? A: the StorageClass's provisioner (CSI driver).
- **4.3** PVC Pending with WaitForFirstConsumer — bug? A: no; expected until a pod consumes it.

## Domain 05 — Troubleshooting
- **5.1** Node NotReady — first three commands? A: `describe node`; on node `systemctl status kubelet` + `journalctl -u kubelet`.
- **5.1** cordon vs drain? A: cordon stops new scheduling; drain also evicts existing pods.
- **5.2** `kubectl` connection refused — where do you go? A: to the node; API server static pod down; check manifest + `crictl logs`.
- **5.2** Edited a static-pod manifest — how to apply? A: just save it; kubelet recreates the pod.
- **5.2** New pods stuck Pending, nodes healthy — component? A: scheduler.
- **5.3** `kubectl top` errors — fix? A: enable metrics-server, wait ~1 min.
- **5.4** CrashLoopBackOff — the one flag? A: `kubectl logs <pod> --previous`.
- **5.4** Exit code 137? A: SIGKILL, usually OOMKilled; raise memory limit.
- **5.4** ImagePullBackOff vs CrashLoop — which is an app bug? A: CrashLoop.
- **5.5** Service returns nothing — fastest first command? A: `kubectl get endpoints <svc>`.
- **5.5** Everything right but blocked in one namespace — forgot? A: a default-deny NetworkPolicy.

## Cross-cutting exam habits (ask these often — they're worth the most points)
- Before any multi-cluster task, what do you do first? A: `kubectl config use-context <name>`.
- Fastest way to author a manifest? A: `kubectl ... --dry-run=client -o yaml > f.yaml`, then edit.
- Don't remember a field name? A: `kubectl explain <resource>.<path>`.
- After doing a task, what always comes next? A: verify with `kubectl get/describe`.
