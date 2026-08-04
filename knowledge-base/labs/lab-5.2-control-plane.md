# Lab 5.2 — Control-Plane (static pod) Troubleshooting

**Lesson:** [[5.2-control-plane-components]] · **Cluster:** single node OK
**Goal:** break a control-plane static pod and recover it by editing the manifest on the node.

> ⚠️ This lab deliberately breaks the API server. That's the point — it's a throwaway cluster.
> Mentor: make a backup first so recovery is guaranteed.

## Part A — Explore the control plane (student)
```bash
kubectl get pods -n kube-system            # see apiserver/etcd/scheduler/controller-manager
minikube ssh
ls -l /etc/kubernetes/manifests/           # the four static-pod manifests
sudo crictl ps | grep -E "apiserver|etcd|scheduler|controller"
exit
```
Ask: "Why can these run before the API server exists?" (kubelet runs static pods from the manifest dir.)

## Part B — Break the scheduler (safe: cluster stays reachable)
```bash
minikube ssh
sudo cp /etc/kubernetes/manifests/kube-scheduler.yaml /tmp/sched.bak
sudo sed -i 's#image: registry.k8s.io/kube-scheduler.*#image: registry.k8s.io/kube-scheduler:v0.0.0#' /etc/kubernetes/manifests/kube-scheduler.yaml
exit
```
Diagnose (student):
```bash
kubectl get pods -n kube-system | grep scheduler     # ImagePullBackOff/Error
kubectl run testpod --image=nginx
kubectl get pod testpod -o wide                       # stuck Pending — no scheduler to place it!
kubectl describe pod testpod                          # Events: no nodes assigned
```
Fix:
```bash
minikube ssh
sudo cp /tmp/sched.bak /etc/kubernetes/manifests/kube-scheduler.yaml
exit
kubectl get pods -n kube-system | grep scheduler     # Running
kubectl get pod testpod -o wide                       # now Scheduled/Running
```

## Part C — Break the API server (the classic) — advanced
```bash
minikube ssh
sudo cp /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/api.bak
sudo sed -i 's/--etcd-servers=/--etcd-servers=https:\/\/BADHOST:2379 #/' /etc/kubernetes/manifests/kube-apiserver.yaml
```
Now `kubectl` (from the host) will start failing with connection refused. Diagnose ON the node:
```bash
sudo crictl ps -a | grep apiserver
sudo crictl logs <apiserver-container-id> | tail -20   # the etcd-connection error
# fix:
sudo cp /tmp/api.bak /etc/kubernetes/manifests/kube-apiserver.yaml
# wait for kubelet to recreate it:
watch sudo crictl ps | grep apiserver
exit
kubectl get nodes                                       # kubectl works again
```

## Verification
- `kubectl get pods -n kube-system` all Running.
- Student can explain: fix static pods by editing the file on the node; kubelet auto-recreates.

## Cleanup
Delete the test pod: `kubectl delete pod testpod`.

## Stretch
- `kubeadm certs check-expiration` (via `minikube ssh sudo ...`) — read cert expiry.
- Temporarily move a manifest out of the dir and watch the component vanish, then move it back.
