# Lab 1.2 — kubeadm Inspection & Rehearsal

**Lesson:** [[1.2-kubeadm-install-prereqs]] · **Cluster:** single node OK
**⚠️ Inspection lab** — minikube is already kubeadm-provisioned, so inspect the real artifacts and
rehearse the bootstrap sequence out loud. You cannot `kubeadm init` a new machine here.

## Part A — Inspect the existing kubeadm cluster (student)
```bash
minikube ssh
# the config kubeadm used:
sudo cat /etc/kubernetes/admin.conf | head -20        # the admin kubeconfig
ls /etc/kubernetes/                                    # admin.conf, kubelet.conf, *.conf, manifests/, pki/
ls /etc/kubernetes/manifests/                          # control-plane static pods (kubeadm wrote these)
ls /etc/kubernetes/pki/                                # certs kubeadm generated
sudo cat /var/lib/kubelet/config.yaml | head -20       # kubelet config
# CRI + CNI evidence:
sudo crictl info | head                                # runtime
ls /etc/cni/net.d/                                     # CNI config
exit
```

## Part B — Rehearse the bootstrap (say it, don't run it)
Have the student narrate, in order, what they'd do on fresh machines. Correct any gaps:
1. Prep each node: install containerd (cgroup driver = systemd), install kubeadm/kubelet/kubectl,
   `swapoff -a`, enable `br_netfilter` + sysctl `bridge-nf-call-iptables=1`, `ip_forward=1`.
2. `sudo kubeadm init --pod-network-cidr=10.244.0.0/16` on the first control-plane node.
3. Set up kubectl: copy `admin.conf` to `~/.kube/config`.
4. Install a CNI (nodes are NotReady until then).
5. `kubeadm join …` on workers using the printed token + CA cert hash.

## Part C — Token practice (this you CAN run)
```bash
minikube ssh
sudo kubeadm token list
sudo kubeadm token create --print-join-command       # the exact worker join command
exit
```
Ask: "A worker's join token expired — what command regenerates it?" (`kubeadm token create --print-join-command`.)

## Verification
- Student can point to where certs, static pods, kubelet config, and CNI config live.
- Student can recite the init→CNI→join order and explain why nodes are NotReady before the CNI.

## Cleanup
None.

## Stretch
- `sudo kubeadm config print init-defaults` (on the node) to see a full init config file.
