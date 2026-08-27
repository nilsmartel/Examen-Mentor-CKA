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
