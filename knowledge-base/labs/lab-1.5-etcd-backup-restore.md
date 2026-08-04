# Lab 1.5 — etcd Backup & Restore (guaranteed exam skill)

**Lesson:** [[1.5-etcd-backup-restore]] · **Cluster:** single node OK. **Fully practicable.**
**Goal:** snapshot etcd, prove the backup captures state, restore it, and repoint the etcd static pod.

> ⚠️ Restore briefly disrupts the control plane. Expected. Throwaway cluster.
> Drill this to fluency — it's one of the most reliable point-earners on the exam.

## Part A — Create known state, then back up (student)
```bash
kubectl create ns backup-test
kubectl -n backup-test create deploy marker --image=nginx    # a thing that should survive
minikube ssh
sudo mkdir -p /var/lib/etcd-backup
sudo ETCDCTL_API=3 etcdctl snapshot save /var/lib/etcd-backup/snap.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
sudo ETCDCTL_API=3 etcdctl snapshot status /var/lib/etcd-backup/snap.db --write-out=table
exit
```
Ask: "Where did you get those three cert paths?" (from `/etc/kubernetes/manifests/etcd.yaml`.)

## Part B — Change state after the backup
```bash
kubectl -n backup-test create deploy after-backup --image=nginx   # created AFTER snapshot
kubectl -n backup-test get deploy                                  # marker + after-backup
```
After restore, `after-backup` should be GONE (it wasn't in the snapshot) — that proves the restore worked.

## Part C — Restore (student, on the node)
```bash
minikube ssh
sudo ETCDCTL_API=3 etcdctl snapshot restore /var/lib/etcd-backup/snap.db \
  --data-dir=/var/lib/etcd-restored
# repoint the etcd static pod at the restored data dir:
sudo sed -i 's#path: /var/lib/minikube/etcd#path: /var/lib/etcd-restored#' /etc/kubernetes/manifests/etcd.yaml
#   ^ NOTE: minikube's etcd hostPath is usually /var/lib/minikube/etcd — CONFIRM first:
grep -n "etcd" /etc/kubernetes/manifests/etcd.yaml   # find the hostPath volume for etcd-data
# the kubelet restarts etcd automatically. Give it ~1-2 min.
exit
```
> Mentor note: the exact hostPath differs by setup. Have the student **read** the manifest and change
> the etcd-data volume's `path` to `/var/lib/etcd-restored`. The lesson's principle (data-dir must
> match manifest) is what matters.

## Part D — Verify the restore
```bash
kubectl get ns backup-test                       # exists
kubectl -n backup-test get deploy                # marker present, after-backup GONE
```
If `after-backup` is gone and `marker` remains → restore succeeded.

## Cleanup
```bash
kubectl delete ns backup-test
```
(Leave the etcd manifest pointing at the restored dir, or restore original path if you prefer a clean state.)

## Stretch
- Time yourself doing the full save+restore in under 5 minutes — exam pace.
