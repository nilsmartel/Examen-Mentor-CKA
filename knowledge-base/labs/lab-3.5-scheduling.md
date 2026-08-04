# Lab 3.5 — Scheduling: Requests, Affinity, Taints

**Lesson:** [[3.5-scheduling-affinity-taints-limits]] · **Cluster:** `--nodes 2` ideal, single node OK
**Goal:** control placement and diagnose Pending caused by requests and taints.

## Part A — Pending from too-large requests (student)
```bash
kubectl run big --image=nginx \
  --overrides='{"spec":{"containers":[{"name":"big","image":"nginx","resources":{"requests":{"cpu":"100"}}}]}}'  # 100 CPUs — won't fit
kubectl get pod big                                     # Pending
kubectl describe pod big | grep -A5 Events              # Insufficient cpu
kubectl delete pod big
```

## Part B — nodeSelector / label a node
```bash
kubectl label node minikube disktype=ssd
kubectl run ssdpod --image=nginx --overrides='{"spec":{"nodeSelector":{"disktype":"ssd"}}}'
kubectl get pod ssdpod -o wide                          # lands on the labeled node
kubectl run nopod --image=nginx --overrides='{"spec":{"nodeSelector":{"disktype":"nvme"}}}'
kubectl get pod nopod                                   # Pending (no node matches)
kubectl describe pod nopod | grep -A5 Events            # didn't match node selector
```

## Part C — Taints & tolerations (break-it/fix-it)
```bash
kubectl taint nodes minikube gpu=true:NoSchedule
kubectl run plain --image=nginx
kubectl get pod plain                                   # Pending (repelled by taint)
kubectl describe pod plain | grep -A5 Events            # untolerated taint
# fix by adding a toleration:
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: Pod
metadata: {name: tolerant}
spec:
  tolerations:
    - {key: gpu, operator: Equal, value: "true", effect: NoSchedule}
  containers: [{name: c, image: nginx}]
EOF
kubectl get pod tolerant -o wide                        # schedules despite the taint
# remove the taint:
kubectl taint nodes minikube gpu=true:NoSchedule-
```
Ask: "Did adding the toleration *attract* the pod, or just permit it?" (permit — attraction needs affinity.)

## Part D — nodeName bypass (scheduler-independent)
```bash
kubectl run pinned --image=nginx --overrides='{"spec":{"nodeName":"minikube"}}'
kubectl get pod pinned -o wide                          # placed directly, no scheduler needed
```

## Verification
- Student induces and reads Pending for both requests and taints, then fixes each.
- Student states taint=repel / toleration=permit / affinity=attract.

## Cleanup
```bash
kubectl delete pod ssdpod nopod plain tolerant pinned --ignore-not-found
kubectl label node minikube disktype-
kubectl taint nodes minikube gpu=true:NoSchedule- 2>/dev/null || true
```

## Stretch
- Add podAntiAffinity to a 3-replica deployment and confirm pods spread across nodes (needs 2 nodes).
