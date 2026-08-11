
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
