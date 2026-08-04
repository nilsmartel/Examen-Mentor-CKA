# Lab 1.7 — Inspect CNI / CSI / CRI

**Lesson:** [[1.7-extension-interfaces-cni-csi-cri]] · **Cluster:** single node OK
**Goal:** locate and identify the active runtime, network plugin, and storage drivers.

## Part A — CRI (student, on the node)
```bash
minikube ssh
sudo crictl info | grep -i runtime          # which runtime (containerd)
sudo crictl ps | head                        # CRI-level container list
sudo crictl images | head
cat /var/lib/kubelet/config.yaml | grep -i cgroup   # cgroupDriver
exit
```

## Part B — CNI
```bash
minikube ssh
ls -l /etc/cni/net.d/                         # CNI config file(s)
cat /etc/cni/net.d/*.conflist 2>/dev/null | head -30
ls /opt/cni/bin/ | head                       # plugin binaries
exit
kubectl get pods -n kube-system -o wide | grep -Ei "cni|calico|flannel|kindnet|cilium"
```
Ask: "If you deleted the CNI config, what would happen to the nodes?" (New pods fail networking;
nodes can go NotReady.)

## Part C — CSI / storage
```bash
kubectl get csidrivers
kubectl get csinodes
kubectl get storageclass
kubectl get sc -o custom-columns=NAME:.metadata.name,PROVISIONER:.provisioner
```
The `PROVISIONER` is the CSI driver (on minikube usually `k8s.io/minikube-hostpath` or a standard one).

## Verification
- Student can state the runtime, the CNI plugin name, and the default StorageClass's provisioner.
- Student can map each: CRI=run containers, CNI=pod networking, CSI=pod storage.

## Cleanup
None.

## Stretch
- `kubectl get pods -n kube-system` and identify which pod implements each interface.
