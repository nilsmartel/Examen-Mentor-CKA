# Lab Index

Every lesson has a matching hands-on lab. All labs assume `minikube start --driver=podman`; the
"Cluster" column flags when a second node or a specific CNI/addon helps. Labs are designed so the
**student types the commands** and always include a break-it/fix-it step where sensible.

| Lab | Lesson | Cluster needs | Fully practicable? |
|-----|--------|---------------|:------------------:|
| lab-1.1-rbac | 1.1 RBAC | single node | ✅ |
| lab-1.2-kubeadm-inspect | 1.2 kubeadm install | single node | 🟡 inspect/rehearse |
| lab-1.3-drain-upgrade | 1.3 lifecycle & upgrades | `--nodes 2` for drain | 🟡 drain ✅ / upgrade rehearse |
| lab-1.4-ha-inspect | 1.4 HA control plane | single node | 🟡 reason/inspect |
| lab-1.5-etcd-backup-restore | 1.5 etcd backup/restore | single node (`minikube ssh`) | ✅ |
| lab-1.6-helm-kustomize | 1.6 Helm & Kustomize | single node | ✅ |
| lab-1.7-interfaces-inspect | 1.7 CNI/CSI/CRI | single node | ✅ (inspect) |
| lab-1.8-crd-operator | 1.8 CRDs & operators | single node | ✅ |
| lab-2.1-pod-connectivity | 2.1 pod connectivity | `--nodes 2` nice | ✅ |
| lab-2.2-services | 2.2 services & endpoints | single node | ✅ |
| lab-2.3-ingress | 2.3 ingress | single node + ingress addon | ✅ |
| lab-2.4-gateway-api | 2.4 Gateway API | single node + CRDs | 🟡 YAML ✅ / controller may vary |
| lab-2.5-network-policies | 2.5 network policies | **`--cni=calico`** to enforce | 🟡 needs policy CNI |
| lab-2.6-coredns | 2.6 CoreDNS | single node | ✅ |
| lab-3.1-rollouts | 3.1 rollouts | single node | ✅ |
| lab-3.2-config-secrets | 3.2 ConfigMaps & Secrets | single node | ✅ |
| lab-3.3-hpa | 3.3 HPA | single node + metrics-server | ✅ |
| lab-3.4-self-healing | 3.4 self-healing | `--nodes 2` for DaemonSet | ✅ |
| lab-3.5-scheduling | 3.5 scheduling | `--nodes 2` nice | ✅ |
| lab-4.1-volumes | 4.1 volumes | single node | ✅ |
| lab-4.2-pv-pvc | 4.2 PV & PVC | single node | ✅ |
| lab-4.3-storageclass | 4.3 StorageClasses | single node | ✅ |
| lab-5.1-node-troubleshooting | 5.1 node troubleshooting | single node (`minikube ssh`) | ✅ |
| lab-5.2-control-plane | 5.2 control-plane | single node (`minikube ssh`) | ✅ |
| lab-5.3-resource-monitoring | 5.3 monitoring | single node + metrics-server | ✅ |
| lab-5.4-logging | 5.4 logging | single node | ✅ |
| lab-5.5-network-troubleshooting | 5.5 net troubleshooting | single node (Part E: calico) | ✅ |

Legend: ✅ = fully doable on minikube · 🟡 = partly (inspect/rehearse, or needs a specific CNI/addon).
Mentor: when a lab is 🟡, be explicit with the student about what is real practice vs rehearsal.
