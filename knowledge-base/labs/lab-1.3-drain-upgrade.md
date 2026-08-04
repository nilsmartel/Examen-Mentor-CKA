# Lab 1.3 — Drain & Upgrade Rehearsal

**Lesson:** [[1.3-cluster-lifecycle-upgrades]] · **Cluster:** `--nodes 2` ideal for drain; single node OK for the rest
**Goal:** practice cordon/drain/uncordon for real; rehearse the upgrade command order.

## Part A — cordon / drain / uncordon (student, real)
Start a 2-node cluster if you can: `minikube start --driver=podman --nodes 2`
```bash
kubectl get nodes
kubectl create deploy web --image=nginx --replicas=4
kubectl get pods -o wide                                  # spread across nodes
# cordon: no NEW pods, existing stay
kubectl cordon minikube-m02
kubectl get nodes                                         # m02 SchedulingDisabled
# drain: evict pods off m02
kubectl drain minikube-m02 --ignore-daemonsets --delete-emptydir-data
kubectl get pods -o wide                                  # web pods rescheduled to the other node
# back into rotation:
kubectl uncordon minikube-m02
```
Ask: "Why did drain need `--ignore-daemonsets`?" (DaemonSet pods can't be evicted normally.)

Single-node fallback: `kubectl cordon minikube` → create a pod → it stays `Pending` → `uncordon` → it schedules.

## Part B — Upgrade command rehearsal (say the order)
Have the student order these correctly for a control-plane node, then a worker:
- Control plane: upgrade `kubeadm` binary → `kubeadm upgrade plan` → `kubeadm upgrade apply vX.Y.Z`
  → `drain` → upgrade kubelet/kubectl → `daemon-reload && restart kubelet` → `uncordon`.
- Worker: `drain` (from a machine with access) → upgrade kubeadm → **`kubeadm upgrade node`** →
  upgrade kubelet/kubectl → `daemon-reload && restart kubelet` → `uncordon`.

Curveball question: "On the worker, is it `kubeadm upgrade apply` or `kubeadm upgrade node`?" (`node`.)

## Part C — Inspect versions
```bash
kubectl get nodes -o wide          # VERSION column = kubelet version per node
kubectl version                    # client + server
```

## Verification
- Student performs cordon→drain→uncordon and explains each.
- Student recites the correct upgrade order and the apply-vs-node distinction without help.

## Cleanup
```bash
kubectl delete deploy web
```

## Stretch
- Drain a node, observe a PodDisruptionBudget block eviction (create a PDB with `minAvailable`).
