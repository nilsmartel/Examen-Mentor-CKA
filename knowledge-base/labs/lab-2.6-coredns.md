# Lab 2.6 — CoreDNS & Cluster DNS

**Lesson:** [[2.6-coredns]] · **Cluster:** single node OK
**Goal:** verify Service/pod DNS names, read resolv.conf, and inspect CoreDNS.

## Part A — Resolve a Service (student)
```bash
kubectl create ns dns
kubectl -n dns create deploy web --image=nginx
kubectl -n dns expose deploy web --port=80
kubectl -n dns run t --image=busybox:1.28 -it --rm -- nslookup web                          # short name
kubectl -n dns run t --image=busybox:1.28 -it --rm -- nslookup web.dns.svc.cluster.local    # FQDN
```
Ask: "Why does the short name work?" (search domains + ndots — see resolv.conf next.)

## Part B — Inspect resolv.conf
```bash
kubectl -n dns run t --image=busybox:1.28 -it --rm -- cat /etc/resolv.conf
# note: nameserver = kube-dns ClusterIP; search = dns.svc.cluster.local svc.cluster.local ...; ndots:5
```

## Part C — Cross-namespace resolution
```bash
kubectl run t --image=busybox:1.28 -it --rm -- nslookup web.dns          # from default -> needs ns
kubectl run t --image=busybox:1.28 -it --rm -- nslookup web              # from default -> FAILS (wrong ns)
```
Point: from another namespace you must qualify with `.dns` (or full FQDN).

## Part D — Inspect CoreDNS itself
```bash
kubectl get svc -n kube-system kube-dns
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl -n kube-system get configmap coredns -o yaml | sed -n '1,40p'    # the Corefile
```

## Part E — Break-it/fix-it (safe): scale CoreDNS to 0
```bash
kubectl -n kube-system scale deploy coredns --replicas=0
kubectl -n dns run t --image=busybox:1.28 -it --rm -- nslookup web       # DNS fails now
kubectl -n kube-system scale deploy coredns --replicas=2                  # restore
```

## Verification
- Student resolves Services by short name and FQDN and explains the search-domain mechanism.
- Student can locate CoreDNS (pods, `kube-dns` svc, Corefile).

## Cleanup
```bash
kubectl delete ns dns
kubectl -n kube-system scale deploy coredns --replicas=2   # ensure restored
```

## Stretch
- Add a `hosts`/rewrite stanza to the CoreDNS Corefile and observe custom resolution (advanced).
