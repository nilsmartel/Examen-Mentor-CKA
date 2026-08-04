# Lab 2.1 — Pod Connectivity

**Lesson:** [[2.1-pod-connectivity]] · **Cluster:** `--nodes 2` ideal (cross-node), single node OK
**Goal:** prove pods reach each other by IP across nodes, and that pod IPs are ephemeral.

## Part A — Pod-to-pod by IP (student)
```bash
kubectl run web --image=nginx
kubectl get pod web -o wide                          # note IP + node
kubectl run client --image=busybox -it --rm -- wget -qO- --timeout=3 http://<web-pod-ip>
```
On a 2-node cluster, confirm they're on different nodes and it still works (rule 2: no NAT across nodes).

## Part B — Ephemeral IPs (the motivation for Services)
```bash
kubectl get pod web -o wide                          # IP #1
kubectl delete pod web
kubectl run web --image=nginx
kubectl get pod web -o wide                          # DIFFERENT IP
```
Ask: "You hardcoded the old IP in a client. What broke, and what fixes it permanently?" (New IP;
a Service gives a stable name/VIP.)

## Part C — Shared network namespace in a pod
```bash
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: Pod
metadata: {name: twocon}
spec:
  containers:
    - {name: web, image: nginx}
    - {name: shell, image: busybox, command: ["sleep","3600"]}
EOF
kubectl exec twocon -c shell -- wget -qO- --timeout=3 http://localhost:80   # reaches nginx via localhost
```
Point: same pod = same netns = `localhost` + same IP; two containers can't bind the same port.

## Verification
- Cross-pod curl by IP works; student can state the 4 network-model rules.
- Student explains why pod IPs shouldn't be relied on.

## Cleanup
```bash
kubectl delete pod web twocon --ignore-not-found
```

## Stretch
- `kubectl get pods -A -o wide` and identify the pod CIDR vs the service CIDR ranges.
