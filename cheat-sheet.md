# Self-Healing Primitives (3.4)

```bash
kubectl explain pod.spec.containers.startupProbe          # discover probe fields (handler + tunables) instead of guessing
kubectl get pod <p> -w                                    # watch: RESTARTS climbs=liveness killing; READY 0/1=readiness gating
kubectl rollout restart daemonset/calico-node -n kube-system  # fix CNI "401 Unauthorized" sandbox failures (stale CNI token)
# THREE PROBES: startup="booted yet?" gates the other two (slow starters) → liveness="alive?" RESTARTS container → readiness="ready?" in/out of endpoints (no restart)
# startup vs liveness = SAME test, different PATIENCE. startup runway = periodSeconds × failureThreshold (set > boot time)
# liveness initialDelaySeconds << boot time ⇒ crash-loop of a HEALTHY app; the fix is startupProbe, not a bigger initialDelay
# probe fields are IMMUTABLE on a running pod ⇒ edit YAML + delete/recreate (kubectl edit rejects the change)
# CONTROLLER PICKER: Deployment/RS=N stateless interchangeable · DaemonSet=1-per-node · StatefulSet=stable identity (needs headless svc clusterIP:None) · Job/CronJob=run-to-completion
```

# CoreDNS & Cluster DNS (2.6)

```bash
kubectl get svc -n kube-system kube-dns                       # the DNS Service (name is 'kube-dns', NOT coredns); ClusterIP ~10.96.0.10
kubectl get pods -n kube-system -l k8s-app=kube-dns           # the CoreDNS pods (Deployment 'coredns', NOT a static pod)
kubectl -n kube-system get configmap coredns -o yaml          # the Corefile (where cluster DNS behavior is tuned)
kubectl run t --image=busybox:1.28 --restart=Never -i --rm -- nslookup <svc>.<ns>.svc.cluster.local  # verify resolution (busybox:1.28!)
kubectl run t --image=busybox:1.28 --restart=Never -i --rm -- cat /etc/resolv.conf                   # nameserver=ClusterIP, search list, ndots:5
kubectl -n kube-system scale deploy coredns --replicas=2      # fix cluster-wide DNS-down (it's a Deployment: scale, don't ssh)
# FQDN: <svc>.<ns>.svc.cluster.local  |  pods: <ip-dashes>.<ns>.pod.cluster.local
# short name works: <5 dots => resolv.conf 'search' appends suffixes; first suffix = querying pod's OWN ns => cross-ns needs <svc>.<ns>
# DNS down EVERYWHERE => CoreDNS itself (pods/kube-dns svc). DNS down for ONE pod => that pod's EGRESS NetworkPolicy blocking :53.
# kube-proxy is NOT a hop: it writes NAT rules into each node's kernel; the kernel netfilter is the dataplane (=> ping ClusterIP fails, curl works)
```

# Network Policies (2.5)

```bash
kubectl -n <ns> get networkpolicy                                   # list policies; a pod NOT selected by any = wide open
kubectl -n <ns> describe networkpolicy <name>                       # read the from/to peers + ports; check dash placement (AND vs OR)
kubectl -n <ns> edit networkpolicy <name>                           # live-fix a policy (rules take effect immediately)
kubectl -n <ns> exec <client> -- wget -qO- --timeout=3 http://<svc> # prove enforcement: allowed=HTML, blocked=timeout
# default-deny ingress: spec.podSelector:{} + policyTypes:[Ingress], no rules  ({} = ALL pods, not none)
# separate '-' items = OR ; podSelector+namespaceSelector under ONE '-' = AND (the exam trap)
# default-deny EGRESS also blocks DNS -> allow egress to kube-system CoreDNS on port 53 (UDP+TCP)
```

# Services & Networking Troubleshooting (5.5)

```bash
kubectl -n <ns> get endpoints <svc>      # FIRST command. EMPTY => selector/label mismatch OR pods not Ready
kubectl -n <ns> get endpointslices -l kubernetes.io/service-name=<svc>  # modern spelling (Endpoints deprecated v1.33+)
kubectl -n <ns> get svc <svc> -o wide    # read the selector; compare to `get pods --show-labels`
kubectl -n <ns> get pods                 # READY col 0/1 = failing readiness probe = dropped from endpoints (Running≠serving)
kubectl -n <ns> run probe --image=busybox --restart=Never -i --rm -- wget -qO- --timeout=3 http://<svc>  # one-shot probe
kubectl -n <ns> get netpol               # LAST hop: a default-deny NetworkPolicy silently drops traffic w/ everything else green
```
```
# Request path: DNS -> Service(ClusterIP) -> EndpointSlice(pod IPs) -> kube-proxy -> Pod -> container port  (NetworkPolicy can block anywhere)
# Checklist: endpoints? -> pods Ready? -> targetPort==container bind port? -> DNS resolves? -> NetworkPolicy?
# targetPort must equal the app's ACTUAL bind port; containerPort/Host Port are cosmetic. Populated eps + refused/timeout => wrong targetPort.
# NetworkPolicy is additive/whitelist & off-until-touched: no policy=allow-all; podSelector:{} + policyTypes:[Ingress] + no rules = deny-all-in.
```

# StorageClasses & Dynamic Provisioning (4.3)

```bash
kubectl get sc                                       # which one is (default)? a PVC with NO storageClassName rides it
kubectl get sc standard -o yaml | grep -E "provisioner|volumeBindingMode|reclaimPolicy"  # the 3 fields that matter
kubectl describe pvc <pvc>                            # Pending? "ExternalProvisioning...verify provisioner running" => provisioner missing/denied
minikube addons list | grep storage                  # on minikube the provisioner is the storage-provisioner ADDON (not built-in)
minikube addons enable storage-provisioner           # no provisioner pod => dynamic PVs never get made
kubectl edit clusterrole system:persistent-volume-provisioner  # WFC needs `nodes get` at cluster scope; add it live (no restart)
```
```
# StorageClass = the TEMPLATE/recipe stamped onto every PV it mints (reclaimPolicy, volumeBindingMode, params). Size comes from the PVC.
# volumeBindingMode:  Immediate = provision PV the instant the PVC exists  |  WaitForFirstConsumer = wait for a POD to be
#   scheduled, THEN provision topology-local to that node. Under WFC a Pending PVC with no pod is NORMAL -> schedule a pod (not a bug).
# WFC makes the provisioner read the target Node (topology) => needs `get nodes`; Immediate never does (that's why the 403 was hidden).
# Dynamic provisioning is PLUGGABLE: needs a live provisioner pod (CSI driver). RBAC is evaluated LIVE per-request; a reconciling
#   controller self-heals the instant you clear the blocker -- no restart, no re-trigger.
# Dynamic PV capacity: EXACT on hostpath; rounds UP on real cloud disks (EBS/PD min granularity).
```

# PersistentVolumes & PVCs (4.2)

```bash
kubectl get pv,pvc                                   # ★ first move -- eyeball STATUS + the STORAGECLASS column on BOTH
kubectl describe pvc <pvc>                            # Pending? Events tell you WHY: "storageclass X not found" vs "no volumes available"
kubectl get pv <pv> -o jsonpath='{.spec.claimRef.name}{"\n"}'  # is it still pinned to a dead claim? (why Released won't rebind)
kubectl edit pv <pv>                                 # delete claimRef: block => Released -> Available (the fix; no patch JSON)
kubectl get pod <p> -o wide                          # NODE column -- a hostPath-backed PV is STILL node-local, data won't travel
# no imperative generator for PV/PVC -- hand-author YAML or crib from kubernetes.io/docs (allowed in exam)
```
```
# BINDING = a 3-field shape match, ALL must hold:  capacity (PV >= request; claim eats the WHOLE PV, no partitioning)
#   + accessModes (PV offers SUPSERSET of requested)  + storageClassName (EXACT string equality).
# storageClassName has THREE states:  omitted = default class (dynamic)  |  "" = static only, matches only ""  |
#   "name" = provisioner reacts IF a StorageClass by that name exists (dynamic), else pure matching key (static).
# Pod names the PVC, never the PV:  volumes: [{persistentVolumeClaim: {claimName: <pvc>}}]
# Node-independence comes from the BACKEND, not the abstraction. hostPath PV = node-bound (no nodeAffinity => scheduler blind).
# Lifecycles are independent: delete POD -> PVC+PV stay Bound. delete PVC (+Retain) -> PV -> Released (data kept, won't rebind).
```

# Volumes, Access Modes & Reclaim (4.1)

```bash
kubectl explain pod.spec.volumes                    # spec.volumes DECLARES once; containers[].volumeMounts REFERENCE by name
kubectl exec shared -c reader -- cat /data/msg      # -c picks the container: proves an emptyDir is shared pod-wide
kubectl get pod hp -o wide                          # ★ NODE column -- run it BEFORE deleting, hostPath lives on that node only
kubectl describe pod broken-mount | tail -15        # ContainerCreating => evidence is in EVENTS, `logs` is impossible
minikube ssh -n minikube-m02 -- sudo ls /mnt/labdata  # -n targets a specific node (default is the `minikube` node)
kubectl edit pv <name>                              # delete the claimRef: block => Released -> Available (no patch JSON needed)
```
```
# TWO LIFECYCLES. A container "restart" = a BRAND-NEW CONTAINER FROM THE IMAGE (that's why each attempt gets its
#   own N.log). Its writable layer is WIPED every time. emptyDir fills the gap: scoped to the POD.
#   emptyDir lives at /var/lib/kubelet/pods/<POD-UID>/volumes/kubernetes.io~empty-dir/<name>
#   ...same pod UID as /var/log/pods/<ns>_<pod>_<POD-UID>/<ctr>/0.log  -> SAME KEY.
#   => survives container restarts (same UID) / dies with the pod (new pod = new UID = guaranteed-empty dir).
#   Use for: crash-survival + SIDECARS (one dir, two containers, no network). `sizeLimit: 500Mi` or it can fill the node.
#
# hostPath = a dir on THE NODE. Survives the pod, does NOT survive a RESCHEDULE. Pods move; the data doesn't.
#   ★ EXAM SCENARIO: "ran fine a week, data vanished overnight, nobody changed anything" => POD MOVED NODES.
#   type: Directory       = an ASSERTION, kubelet verifies it exists -> missing => stuck ContainerCreating/FailedMount
#   type: DirectoryOrCreate = create it if absent
#   Security: a pod mounting hostPath /etc/kubernetes reads admin.conf and owns the cluster. Hence restricted.
#
# ACCESS MODES ARE NODE-SCOPED (except one!) -- matching attribute between PVC (shopping list) and PV (inventory):
#   RWO  ReadWriteOnce     r/w by a single NODE   <- 3 replicas on the SAME node all mount it FINE. "Once" != one pod.
#   ROX  ReadOnlyMany      read-only, many nodes
#   RWX  ReadWriteMany     r/w many nodes -- needs an NFS/CephFS-class backend, NOT any storage
#   RWOP ReadWriteOncePod  r/w by a single POD    <- the only pod-scoped one; exists because RWO is so misread
#
# RECLAIM POLICY (on the PV) IS EVENT-DRIVEN, NOT GARBAGE COLLECTION. Fires when a BOUND PVC is deleted.
#   An Available PV that was never claimed is NEVER swept, however long it sits.
#   Delete = PV + backing storage destroyed (usual default for dynamic provisioning)
#   Retain = data kept, PV -> `Released`, A DELIBERATE DEAD END: it will NOT rebind however well it matches.
#     Why: binding is SHAPE-MATCHING (class + accessModes + capacity), so auto-rebinding would hand
#     team A's payroll data to team B. A human must decide. (`Recycle` is deprecated -- ignore it.)
#   ★ TELL-TALE PAIR: `PVC Pending` + `PV Released` -> stale binding. Fix = clear claimRef.
#   Pinning levers (opt-in, vs default shape-matching): PVC.spec.volumeName / PV.spec.claimRef / PVC.spec.selector
#
# POD SPECS ARE ALMOST ENTIRELY IMMUTABLE. Can't edit volumes/env/resources on a live pod -- apiserver rejects it.
#   Mutable short-list ~= image, activeDeadlineSeconds, adding tolerations.
#   => to change a volume: DELETE + RECREATE (bare pod), or edit the TEMPLATE and let the rollout replace pods.
#   PVs are not pods -- `kubectl edit pv` works fine.
#
# volumeMode: Filesystem (default) | Block (raw device)
# Docs: /docs/concepts/storage/volumes/ and /docs/concepts/storage/persistent-volumes/
```

# Autoscaling / HPA (3.3)

```bash
kubectl autoscale deploy php --cpu=50% --min=1 --max=5   # % => Utilization (NEEDS requests). --cpu-percent is DEPRECATED
kubectl autoscale deploy php --cpu=500m --min=1 --max=5  # quantity => AverageValue (absolute, needs NO requests)
kubectl set resources deploy php --requests=cpu=100m --limits=cpu=200m  # the fix for <unknown>; edits the pod TEMPLATE -> rollout
kubectl get hpa php -w                     # ★ read the TARGETS column: "<unknown>/50%" = decorative HPA, never scales
kubectl describe hpa php | tail -10        # Events log every decision + its reason. Where you debug an HPA
kubectl config set-context --current --namespace=hpa-fun  # saves typing -n; also a fail-silent trap (check get-contexts)
```
```
# HPA = more/fewer PODS.  VPA = bigger/smaller pods (right-sizing requests).  Cluster Autoscaler = more/fewer NODES.
# THE FRACTION:  utilization = actual / REQUESTED.  "50% CPU" means "half of what the pod asked for", not half a core.
#   => resources:{} means the DENOMINATOR IS ZERO => TARGETS "<unknown>" => HPA silently never scales. FAIL-SILENT:
#      `kubectl autoscale` SUCCEEDS, the object looks fine in `get hpa`, and nothing ever happens.
#   <unknown> has exactly TWO causes: (1) no metrics-server / dead metrics API   (2) pods have no cpu REQUESTS
# THE MATH:  desiredReplicas = ceil( currentReplicas x currentMetric / targetMetric )
#   Computed in ONE SHOT, not incrementally. 1 pod at 250% with target 50% => ceil(1 x 250/50) = 5 => jumps 1->5 at once.
#   (rate limiter for huge jumps: won't more than double, or add 4 pods, per 15s)
# TWO TARGET TYPES (autoscaling/v2 `target.type`):
#   Utilization  (--cpu=50%)  = ratio, needs requests   | AverageValue (--cpu=500m) = absolute, needs nothing
#   BUT still set requests anyway: no requests => BestEffort => invisible to the scheduler + evicted first.
# WHY IT SEEMS SLOW (30-60s is NORMAL, not a fault): metrics-server scrapes every 15s + CPU is a rate (needs 2 scrapes)
#   + HPA controller re-evaluates every 15s.
# ASYMMETRY: scale UP fast (seconds) / scale DOWN waits a 5-MINUTE stabilization window (takes the max over the window).
#   Deliberate: prevents flapping. Scaling up is cheap insurance; scaling down bets the load is really gone.
#   Trap: "replicas didn't drop, it's broken" -> no, you're inside the window.
# ★ A RECONCILING LOOP ALWAYS BEATS A ONE-SHOT IMPERATIVE WRITE. `kubectl scale` on an HPA-managed deploy is
#   overwritten within ~15s. Same pattern as: RS vs an edited live pod (3.1), kubelet vs an edited static pod (5.2).
#   HPA and the Deployment controller are LAYERED, not rivals: HPA writes spec.replicas, the Deployment obeys it.
```
# Resource Usage Monitoring (5.3)

```bash
minikube addons enable metrics-server      # NOT installed by default. "addons" + "enable" — both words exact
kubectl top nodes                          # CPU(cores) in millicores + % OF ALLOCATABLE (not of 1000m!)
kubectl top pods -A --sort-by=cpu | head   # ★ FIND THE HOG — -A is the question; a hog hides on the node you didn't check
kubectl top pod <pod> -n <ns> --containers # per-container breakdown (PLURAL = show all; -c/--container = pick one)
kubectl describe node <node>               # read the bottom: "Allocated resources" = REQUESTS (grep -A6 "Allocated resources", lowercase r)
kubectl get apiservice v1beta1.metrics.k8s.io   # ★ a Running pod ≠ a working API. This names the real failure
```
```
# THE TWO LEDGERS — the whole lesson. They are independent and BOTH are true:
#   requests = a RESERVATION. The scheduler's ONLY currency. It never calls the metrics API.   -> describe node
#   limits   = a CEILING enforced by the kubelet.  CPU over -> THROTTLED. Memory over -> OOMKILLED.
#   top      = ACTUAL consumption right now.                                                    -> kubectl top
# => "node is 15% idle but pod is Pending/Insufficient cpu" = requests are committed, not consumed. Hotel: rooms BOOKED, guests absent.
# => a `kubectl run` pod has resources:{} => QoS BestEffort => charged 0 => INVISIBLE to the scheduler
#    (it can burn a full core while its node reads 0% allocated — and the scheduler keeps packing pods onto it)
# You may overcommit LIMITS. You may NEVER overcommit REQUESTS (hard-capped at allocatable).
# top node != sum of top pods — the node figure includes OS + kubelet + containerd + kernel.
# capacity vs allocatable: scheduler only ever spends ALLOCATABLE (capacity - kube/system-reserved - eviction headroom).
# 1000m = 1 core (absolute). "%" = share of THAT NODE's allocatable (5-core node: 1000m shows as 20%).
# CPU is compressible (throttle, survives); memory is not (OOMKill, dies).
# NO metrics-server => kubectl top errors "Metrics API not available" AND HPA reports <unknown> and never scales.
# kubectl is FORGIVING on resource types (node=nodes=no) and STRICT on subcommands/flags (logs, addons, --containers).
```

# Container Output & Logging (5.4)

```bash
kubectl logs <pod> --previous              # ★ CRASHLOOP GOLD — the corpse's output. Run it FIRST, logs get GC'd
kubectl logs <pod> -c <container>          # multi-container / init containers (else "choose a container" error)
kubectl logs <pod> -f --tail=50            # follow; kubectl logs -l app=web --all-containers  (by label)
kubectl describe pod <pod>                 # Events at the bottom + Last State (Reason / Exit Code)
kubectl get events -n <ns> --sort-by=.lastTimestamp   # get events is UNSORTED by default — always pass --sort-by
kubectl exec -it <pod> -- sh               # when the app logs to a FILE inside the container, logs is empty by design
# TWO SEPARATE STREAMS — never conflate:
#   journalctl -u kubelet  = the NODE AGENT's own output (mount failures, sandbox errors)  [ssh + sudo]
#   kubectl logs           = YOUR APP's stdout/stderr of PID 1 only
# WHY --previous EXISTS: runtime writes /var/log/pods/<ns>_<pod>_<uid>/<ctr>/0.log, 1.log, 2.log — ONE FILE PER RESTART.
#   plain `logs` reads the NEWEST (a 4-second-old container that hasn't errored yet). --previous reads N-1.log.
#   The kubelet GCs dead containers → you get ONE step back, and it has a shelf life. Act fast.
#   "previous terminated container not found" = an ERROR, not empty = this pod has NEVER restarted (diagnostic!)
# EVIDENCE-LOCATION MAP (which tool, per symptom):
#   Pending            -> EVENTS  (Node: empty = scheduler's problem; no container ever existed, logs IMPOSSIBLE)
#   ImagePullBackOff   -> EVENTS  (never executed a line of your code — registry/tag/secret, NOT an app bug)
#   CrashLoopBackOff   -> logs --previous   (app ran, then failed — the only one that IS an app bug)
#   OOMKilled          -> describe → Last State
#   Running but broken -> logs + exec + probe config
# "-BackOff" SUFFIX = the kubelet's retry RATE-LIMITER, never a root cause. Strip it, ask what actually failed.
#   ErrImagePull -> ImagePullBackOff is the same disease at a later stage.
# EXIT CODES are UNIX: killed by signal N => 128+N.  137=128+9 SIGKILL   143=128+15 SIGTERM   139=segfault
#   ★ Reason and Exit Code are INDEPENDENT. Trust `Reason: OOMKilled` (authoritative, read from the memory cgroup).
#     137 is a hint, not a requirement — an OOM-killed CHILD process can leave PID 1 exiting with plain 1.
#   137 + Reason:OOMKilled = raise resources.limits.memory.  137 during a delete = grace period (30s) expired, normal.
#   Bare pod resources aren't editable in place -> delete & recreate. Deployment -> edit the pod TEMPLATE.
```

# kubeconfig — what kubectl actually reads (1.2)

```bash
mkdir -p $HOME/.kube && sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config   # kubeadm leaves it root-owned 600
sudo chown $(id -u):$(id -g) $HOME/.kube/config                 # WITHOUT this it stays unreadable to your user
sudo kubectl --kubeconfig=/etc/kubernetes/admin.conf get nodes  # ★ on an ssh'd node you have NO kubeconfig — this is the fix
kubectl config view --minify                                    # what am I actually pointed at right now?
kubectl config get-contexts && kubectl config use-context <ctx>  # ALWAYS switch context first (exam point-loser #1)
# "connection refused to localhost:8080" = kubectl found NO config and fell back to a guess (not a dead apiserver)
# A kubeconfig is mTLS serialized to YAML — same shape as the etcdctl flags and as kubelet.conf:
#   clusters: server URL + certificate-authority-data   -> how I VERIFY the apiserver   (one to check them)
#   users:    client-certificate-data + client-key-data -> who I am + proof             (two to prove yourself)
#   contexts: cluster + user + namespace
# admin.conf gets its power via an RBAC binding → mangled RBAC can lock admin.conf out too.
# super-admin.conf (1.29+) is the break-glass credential that still works. Both in /etc/kubernetes/.
```

# kubeadm Bootstrap & TLS Node Join (1.2)

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16    # writes CP static-pod manifests + certs + bootstrap token
kubectl apply -f <cni-manifest>                        # kubeadm does NOT install a CNI → nodes stay NotReady until you do
sudo kubeadm token create --print-join-command          # regenerate a lost/expired join cmd (default token TTL = 24h)
sudo kubeadm token list                                 # TTL / EXPIRES / EXTRA GROUPS columns
kubectl get csr && kubectl certificate approve <csr>    # a stuck join surfaces as a PENDING CSR
kubectl get secrets -n kube-system --field-selector type=bootstrap.kubernetes.io/token  # tokens ARE Secrets: bootstrap-token-<id>
# TWO OPPOSITE TRUST DIRECTIONS in the join command:
#   --token                          cluster trusts NODE: 24h Secret, group system:bootstrappers:kubeadm:default-node-token,
#                                    whose ENTIRE RBAC power is "may create a CSR". Delete the Secret = instant revoke.
#   --discovery-token-ca-cert-hash   node trusts CLUSTER: the CA cert is PUBLIC but BULKY — the hash solves TRANSPORT,
#                                    not secrecy. Node pulls the CA anonymously from the cluster-info ConfigMap in
#                                    kube-public, hashes it, compares. Same pattern as an SSH host-key fingerprint.
# FLOW: bootstrap-kubelet.conf (token) → CSR → auto-approve → /var/lib/kubelet/pki/ → kubelet.conf
#       → identity becomes system:node:<name> in group system:nodes (Node authorizer scopes it to that node only)
# INIT ORDER: prep every box (containerd + cgroupDriver=systemd, kubeadm/kubelet/kubectl, swapoff -a,
#             br_netfilter + bridge-nf-call-iptables=1) → init → cp admin.conf ~/.kube/config → CNI → join
# TWO FILES, don't confuse: /etc/kubernetes/kubelet.conf = IDENTITY (kubeconfig)
#                           /var/lib/kubelet/config.yaml = BEHAVIOR (staticPodPath, cgroupDriver, clusterDNS)
# CERTS: /etc/kubernetes/pki/ on real kubeadm (minikube deviates: /var/lib/minikube/certs).
#        Don't guess — grep the static-pod manifest: grep ca-file /etc/kubernetes/manifests/kube-apiserver.yaml
# kubeadm upgrade apply = CLUSTER-scoped, run ONCE. drain→pkg→restart→uncordon = PER-MACHINE, on EVERY node incl. CP.
```


# Kubelet Package Upgrade — step 5 deep-dive (1.3)

```bash
# 0. FIRST edit the repo — pkgs.k8s.io is PER-MINOR, v1.34 repo will never offer 1.35
sudo vi /etc/apt/sources.list.d/kubernetes.list   # .../core:/stable:/v1.35/deb/  <- bump the minor
sudo apt-mark unhold kubelet kubectl              # release the pin (hold blocks routine apt upgrade)
sudo apt-get update && sudo apt-get install -y kubelet=1.35.1-1.1 kubectl=1.35.1-1.1  # pin EXACT version
sudo apt-mark hold kubelet kubectl                # re-pin immediately
sudo systemctl daemon-reload && sudo systemctl restart kubelet  # new binary on disk != upgraded process
apt-cache policy kubelet                          # DEBUG "version not found": shows versions + SOURCE REPO
# hold guards TIMING (patch drift, un-drained restart); per-minor repo guards MAGNITUDE (minor jumps)
# hold is what makes routine `apt upgrade` for OS CVEs safe on a k8s node
# /var/lib/kubelet/config.yaml is NOT package content — kubeadm writes it at init/join
```

# Stuck Pod Status → Cause Triage (5.1/5.2 cross-cutting)

```bash
kubectl describe pod <p> | grep -i '^Node:'   # EMPTY = scheduler's problem / ASSIGNED = kubelet's problem
# Pending            -> not scheduled: requests too big, taint, affinity/nodeSelector, unbound PVC, scheduler down
# ContainerCreating  -> CNI can't assign IP (most common), volume won't mount, image still pulling
# ErrImagePull       -> bad tag, missing imagePullSecret, no registry route
# CreateContainerConfigError -> referenced ConfigMap/Secret does not exist
# CrashLoopBackOff   -> app crashes: kubectl logs --previous
# Running 0/1 Ready  -> readiness probe failing (this is the empty-endpoints cause)
kubectl logs <p> --previous                   # the corpse holds the evidence (cf. crictl ps -a)
# lifecycle spine: schedule -> pull -> wire network -> run -> ready
```

# Cordon / Drain / Uncordon (1.3, hands-on)

```bash
kubectl cordon <node>                                          # SchedulingDisabled: no NEW pods, existing stay
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data # cordon + EVICT; flags = waivers you must sign
kubectl uncordon <node>                                        # schedulable again; running pods do NOT rebalance back
kubectl get pods -o wide                                       # verify where pods landed (drain moves movable pods off)
# --ignore-daemonsets: DS pods are pinned per-node → evicting = pointless (recreated instantly)
# --delete-emptydir-data: emptyDir is node-local scratch → eviction DESTROYS it, so drain makes you acknowledge
```

# Cluster Upgrades — kubeadm order (1.3, concept intro)

```bash
# RULES: one minor at a time (1.34→1.35, never skip); control plane FIRST, then workers
# --- Control-plane node ---
sudo kubeadm upgrade plan                              # shows target version
sudo kubeadm upgrade apply v1.35.x                     # CP uses 'apply' — upgrades control-plane static pods
kubectl drain <cp-node> --ignore-daemonsets            # then upgrade kubelet+kubectl pkgs
sudo systemctl daemon-reload && sudo systemctl restart kubelet   # restart after installing new kubelet
kubectl uncordon <cp-node>
# --- Worker node (run kubectl from a box with cluster access) ---
kubectl drain <worker> --ignore-daemonsets --delete-emptydir-data
sudo kubeadm upgrade node                              # WORKER uses 'node', NOT 'apply' (syncs to CP's decision)
kubectl get nodes -o wide                              # VERSION col = kubelet version per node
```

# Node Troubleshooting — kubelet break/fix (5.1 lab)

```bash
kubectl get nodes; kubectl describe node minikube     # NotReady? Conditions all Unknown = frozen heartbeat (not False!)
minikube ssh -- 'sudo systemctl status kubelet'       # dead=inactive(dead) → just stopped;  activating/failed → crash-looping on bad config
minikube ssh -- 'sudo journalctl -u kubelet | tail -20' # READ the error: "yaml: line N" = parse error in /var/lib/kubelet/config.yaml (file present, contents bad — NOT missing)
sudo vi /var/lib/kubelet/config.yaml; sudo systemctl restart kubelet  # fix the file, then explicit restart (Restart=always self-heals but don't wait); daemon-reload only if you edit a unit FILE
kubectl drain minikube --ignore-daemonsets --delete-emptydir-data     # cordon(=SchedulingDisabled, no NEW pods) + EVICT existing; uncordon to restore
```

# Node Health / NotReady (5.1 primer)

```bash
kubectl describe node <node>                          # read Conditions + Events: WHY NotReady (Ready condition = driven by kubelet heartbeat/Lease)
kubectl get leases -n kube-node-lease                 # the heartbeat objects the node-controller (in kcm) watches; silence => NotReady after ~40s, evict after ~5m
minikube ssh; systemctl status kubelet                # kubelet is a SYSTEMD SERVICE (not a static pod) — first stop when a node goes NotReady
journalctl -u kubelet -f                              # the real reason kubelet is unhealthy/can't reach apiserver
sudo systemctl restart kubelet                        # after fixing config/certs; node re-posts heartbeat => Ready again
```

# systemd: systemctl & journalctl (5.1 toolkit)

```bash
systemctl status kubelet        # the VERDICT: Active: active(running)/inactive(dead)/failed/activating(auto-restart) + Loaded: enabled?/unit path + last ~10 log lines
systemctl start|stop kubelet    # affects it RIGHT NOW, this boot only
systemctl enable|disable kubelet# affects NEXT-BOOT autostart (orthogonal axis!); `enable --now` = both at once
systemctl restart kubelet       # stop+start (kubelet needs full restart to re-read config)
journalctl -u kubelet           # the EVIDENCE: full log stream for ONE unit. flags: -f follow, -e end, -p err errors-only, --since "5 min ago", -b this-boot
sudo systemctl daemon-reload    # MUST run after editing any unit/drop-in file, else restart uses STALE config (the systemd cousin of the static-pod trap)
# kubelet startup flags come from the kubeadm drop-in: /etc/systemd/system/kubelet.service.d/10-kubeadm.conf (--config=/var/lib/kubelet/config.yaml). break = file missing OR wrong path in drop-in
```

# Control-Plane / Static-Pod Troubleshooting (5.2)

```bash
ls /etc/kubernetes/manifests/                       # the 4 control-plane static pods (apiserver/etcd/scheduler/controller-manager); kubelet runs these from FILE
minikube ssh; sudo vi /etc/kubernetes/manifests/kube-apiserver.yaml   # FIX a static pod = edit the file on the node; kubelet auto-recreates (NO kubectl apply; kubectl edit won't stick)
sudo crictl ps -a --name kube-apiserver             # API server DOWN => kubectl dead => go to node; -a shows the CRASHED container (evidence is in the corpse); needs sudo
sudo crictl logs <container-id> 2>&1 | tail -30     # the real fatal error (e.g. can't reach etcd -> "F ... context deadline exceeded")
kubectl get pods -A | grep -v Running               # symptom map: apiserver=kubectl refused / scheduler down=new pods stuck Pending / controller-mgr=no self-heal / etcd=apiserver won't start
# after recovering apiserver, wait 30-60s: downstream CrashLoops (storage-provisioner) are AFTERMATH, not root cause — fix the one real thing
```

# etcd Backup & Restore (1.5)

```bash
grep -E "cert|key|listen-client|data-dir" /etc/kubernetes/manifests/etcd.yaml   # READ cert paths + endpoint from the manifest (never guess)
ETCDCTL_API=3 etcdctl snapshot save /var/lib/etcd-backup/snap.db --endpoints=https://127.0.0.1:2379 --cacert=<ca.crt> --cert=<server.crt> --key=<server.key>   # SAVE needs 3 certs (live mTLS)
etcdutl snapshot status /path/snap.db --write-out=table       # verify a backup (hash/revision/keys) — 3.6: on etcdutl
etcdutl snapshot restore /path/snap.db --data-dir=/var/lib/etcd-restored   # RESTORE is OFFLINE => NO cert flags; builds a NEW data dir
# then edit /etc/kubernetes/manifests/etcd.yaml -> change ONLY the etcd-data volume hostPath.path to the restored dir (leave --data-dir flag + mountPath); kubelet restarts etcd
grep staticPodPath /var/lib/kubelet/config.yaml              # how to LOCATE the manifest dir if you blank (kubelet config -> staticPodPath)
# minikube-only: host lacks etcdctl + distroless image => wrap: kubectl -n kube-system exec etcd-minikube -- etcdctl/etcdutl ... (write only to mounted paths /var/lib/minikube/etcd|certs)
```

# RBAC (1.1)

```bash
kubectl create role pod-reader --verb=get,list,watch --resource=pods -n web        # namespaced Role (auto-resolves apiGroup!)
kubectl create rolebinding jane-reader --role=pod-reader --user=jane -n web         # bind Role to a User in one ns
kubectl create clusterrole cm-reader --verb=get,list --resource=configmaps          # ClusterRole (no -n)
kubectl create rolebinding cm-web --clusterrole=cm-reader --user=jane -n web        # ClusterRole pinned to ONE ns via RoleBinding
kubectl create sa reader-sa -n web && kubectl create rolebinding sa-r --role=pod-reader --serviceaccount=web:reader-sa -n web
kubectl auth can-i list pods -n web --as=jane                                       # impersonate USER (--as, NOT --user)
kubectl auth can-i list pods -n web --as=system:serviceaccount:web:reader-sa        # impersonate an SA
kubectl auth can-i --list --as=jane -n web                                          # enumerate everything jane can do
kubectl get pods -n web --as=jane                                                   # real request => 403 NAMES the group/resource/ns
# nodes/PVs/namespaces are cluster-scoped => MUST be ClusterRole+ClusterRoleBinding | binding sets scope, role sets powers
# verbs: get list watch create update patch delete deletecollection (NO "edit") | core group="" , deployments=apps
```

# Services & Endpoints (2.2)

```bash
kubectl expose deploy web --port=80 --target-port=80                    # ClusterIP over a deployment's pods
kubectl expose deploy web --name=web-np --type=NodePort --port=80       # NodePort (adds 3xxxx on every node)
kubectl get ep web                                                      # #1 debug: empty => selector/readiness; IPs w/ wrong port => targetPort
kubectl get endpointslices -l kubernetes.io/service-name=web            # slices named <svc>-<rand>, find by label not name
kubectl patch svc web -p '{"spec":{"selector":{"app":"web"}}}'         # fix a broken selector
kubectl api-resources | grep -i policy                                  # find a resource's short name / group fast
# port=client-facing svc port | targetPort=pod's real listen port (must match app) | nodePort=external 3xxxx
# headless: clusterIP:None => no VIP, DNS returns pod IPs; + StatefulSet => web-0.web.ns.svc.cluster.local
```

# Pod Connectivity & Network Model (2.1)

```bash
kubectl get pods -A -o wide                                              # pod IPs + which node each is on
kubectl run client --image=busybox -it --rm -- wget -qO- --timeout=3 http://<pod-ip>   # test pod-to-pod by IP
kubectl exec twocon -c shell -- wget -qO- http://localhost:80           # same pod = same netns, reach via localhost
kubectl logs <pod> -c <container>                                       # e.g. spot "Address already in use" port clash
# no CNI -> nodes NotReady; CNI broken -> pods ContainerCreating; fix = apply/repair CNI manifest
# pod CIDR e.g. 10.244.0.0/16 (real pod IPs) | service CIDR e.g. 10.96.0.0/12 (virtual, kube-proxy-translated)
```
