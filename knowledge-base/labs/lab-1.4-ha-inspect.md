# Lab 1.4 — HA Control Plane (inspection & reasoning)

**Lesson:** [[1.4-ha-control-plane]] · **Cluster:** single node OK
**⚠️ Reasoning lab** — you can't build a 3-CP HA cluster on this laptop. Inspect topology and drill quorum math.

## Part A — See the single point of failure (student)
```bash
kubectl get pods -n kube-system -o wide | grep -E "etcd|apiserver"   # one of each — no HA
kubectl get nodes                                                     # one control-plane node
```
Ask: "If this node dies, what happens to the cluster?" (Full control-plane outage; workloads keep
running but you can't manage them.)

## Part B — Quorum math drill (whiteboard, no cluster)
Fill in together:
| etcd members | quorum | failures tolerated |
|:---:|:---:|:---:|
| 1 | ? | ? |
| 3 | ? | ? |
| 5 | ? | ? |
| 4 | ? | ? |
(Answers: 1→1/0, 3→2/1, 5→3/2, 4→3/1 — same tolerance as 3, so use odd numbers.)

## Part C — Topology reasoning
Have the student explain in one line each:
- **Stacked etcd** vs **external etcd** (where etcd runs).
- The role of the **load balancer** in front of the API servers, and what `--control-plane-endpoint` sets.
- The extra flags to join an *additional control-plane* node: `--control-plane --certificate-key`.

## Part D — Inspect etcd membership (this you can run)
```bash
minikube ssh
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
exit
```
Single member here — in HA you'd see 3 or 5.

## Verification
- Student produces the quorum table from memory and justifies odd member counts.
- Student explains stacked vs external and the LB's purpose.

## Cleanup
None.

## Stretch
- Read `kubernetes.io/docs/.../ha-topology/` and sketch both topologies.
