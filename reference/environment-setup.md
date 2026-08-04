# Environment Setup — minikube + podman on macOS

Reference for the mentor. The practice cluster is **minikube driven by podman**. Minikube's nodes
are provisioned with **kubeadm**, which makes this a surprisingly complete CKA lab environment.

## Prerequisites (already installed on this Mac)

| Tool | Verify | Notes |
|------|--------|-------|
| podman | `podman version` | Needs a running machine: `podman machine list` |
| minikube | `minikube version` | Driver = podman |
| kubectl | `kubectl version --client` | Client is what matters locally |
| helm | `helm version` | For lesson 1.6 and Helm-based labs |

## Start / stop

```bash
# 0. Make sure the podman machine is running (rootless is fine and recommended)
podman machine list
podman machine start            # if not already running

# 1. Single-node cluster (default for most labs)
minikube start --driver=podman

# 2. Multi-node (scheduling, network policy, node-troubleshooting labs)
minikube start --driver=podman --nodes 2

# Housekeeping
kubectl get nodes               # health check before any lab
minikube status
minikube stop                   # keep the cluster, free resources
minikube delete                 # nuke it (fresh start)
```

**macOS caveat:** use the **podman driver only** — do *not* add `--container-runtime=cri-o`
(CRI-O isn't available on macOS). The default runtime is fine. If podman is rootful you may need
`minikube start --driver=podman --container-runtime=containerd`; try the plain command first.

If `minikube start` misbehaves, `minikube delete && minikube start --driver=podman` resolves most
first-run issues. Give the VM enough resources for multi-node: `--cpus 2 --memory 4096` per your Mac.

## Addons you'll enable during labs

```bash
minikube addons list
minikube addons enable metrics-server   # lessons 3.3 (HPA) and 5.3 (kubectl top)
minikube addons enable ingress          # lesson 2.3 (nginx ingress controller)
```

## Reaching Services / Ingress from the host (macOS gotcha)

On the podman/docker driver on macOS the **node IP (`minikube ip`) is not routable from your Mac**, so
`curl <nodeIP>:<port>` and `curl $(minikube ip)` won't work. Use one of:
- `kubectl port-forward svc/<svc> 8080:80` then `curl 127.0.0.1:8080` (works for any Service/Ingress
  controller — the most reliable option in labs).
- `minikube service <svc> --url` (opens a tunnel and prints a reachable `127.0.0.1:<port>` URL).
- `minikube tunnel` (needs sudo; required to give `LoadBalancer` Services an external IP).

## Getting onto a node (the key to admin/troubleshooting labs)

Because nodes are kubeadm-based, `minikube ssh` is a real CKA-style node shell:

```bash
minikube ssh                      # single node
minikube ssh -n minikube-m02      # a specific node in a multi-node cluster

# Once inside a control-plane node:
ls /etc/kubernetes/manifests/     # static pod manifests: etcd, apiserver, controller-mgr, scheduler
sudo crictl ps                    # containers via the CRI (kubeadm-style)
sudo systemctl status kubelet     # the kubelet (troubleshooting labs stop/start this)
sudo journalctl -u kubelet -f     # kubelet logs
cat /var/lib/kubelet/config.yaml  # kubelet config
```

etcd runs as a static pod, so `etcdctl` backup/restore (lesson 1.5) is fully practicable from inside
the node. The etcd certs live under `/etc/kubernetes/pki/etcd/`.

## What you CAN and CANNOT fully practice here

**Fully practicable on minikube:**
- RBAC, workloads, scheduling, storage (hostPath/dynamic), services, ingress, network policies,
  CoreDNS, CRDs/operators, Helm/Kustomize.
- etcd snapshot **save** and **restore**; inspecting/repairing static pods; kubelet failures;
  `kubectl top`; node cordon/drain; a **single-node** "upgrade" walkthrough.

**Concept-heavy (inspect + rehearse, not a full build):**
- **kubeadm install from scratch** (1.2) — you can inspect the existing kubeadm node, read
  `kubeadm config`, and rehearse the `kubeadm init`/`join`/token/CNI sequence, but not bootstrap a
  brand-new multi-machine cluster here.
- **True HA control plane** (1.4) — inspect the topology and load-balancer concept; can't stand up
  3 stacked-etcd control-plane nodes on this laptop realistically.
- For these, supplement with the free **killercoda.com** CKA scenarios if the student wants a real
  kubeadm sandbox. (Optional — not required to learn the material.)

## Cleanup discipline

After each lab, remove what it created so the next lab starts clean:
```bash
kubectl delete -f <file>            # or
kubectl delete ns <lab-namespace>   # labs use a dedicated namespace where practical
```
