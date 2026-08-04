# Lab 4.2 — PV & PVC

**Lesson:** [[4.2-pv-pvc]] · **Cluster:** single node OK
**Goal:** bind a static PV/PVC, mount it, prove persistence, and diagnose a Pending PVC.

## Part A — Static PV + PVC bind (student)
```bash
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: PersistentVolume
metadata: {name: pv-manual}
spec:
  capacity: {storage: 1Gi}
  accessModes: [ReadWriteOnce]
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath: {path: /mnt/pvdata, type: DirectoryOrCreate}
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata: {name: pvc-manual}
spec:
  accessModes: [ReadWriteOnce]
  storageClassName: manual
  resources: {requests: {storage: 500Mi}}
EOF
kubectl get pv,pvc            # both should show Bound
```

## Part B — Mount it & prove persistence
```bash
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: Pod
metadata: {name: user}
spec:
  containers:
    - name: c
      image: busybox
      command: ["sh","-c","echo durable > /data/f; sleep 3600"]
      volumeMounts: [{name: d, mountPath: /data}]
  volumes:
    - {name: d, persistentVolumeClaim: {claimName: pvc-manual}}
EOF
kubectl exec user -- cat /data/f       # durable
kubectl delete pod user
# recreate a reader pod on the same PVC:
kubectl run reader --image=busybox -it --rm --overrides='{"spec":{"volumes":[{"name":"d","persistentVolumeClaim":{"claimName":"pvc-manual"}}],"containers":[{"name":"r","image":"busybox","stdin":true,"tty":true,"command":["cat","/data/f"],"volumeMounts":[{"name":"d","mountPath":"/data"}]}]}}'
# -> prints 'durable' : data survived the pod
```

## Part C — Break-it/fix-it: Pending PVC (class mismatch)
```bash
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: PersistentVolumeClaim
metadata: {name: pvc-bad}
spec:
  accessModes: [ReadWriteOnce]
  storageClassName: nonexistent
  resources: {requests: {storage: 500Mi}}
EOF
kubectl get pvc pvc-bad                 # Pending
kubectl describe pvc pvc-bad            # no matching PV / class
kubectl delete pvc pvc-bad
```
Ask: "Name the three attributes a PVC and PV must agree on to bind." (capacity, access modes, class.)

## Verification
- PV/PVC Bound; data survives pod deletion; student can explain a Pending PVC.

## Cleanup
```bash
kubectl delete pod user reader --ignore-not-found
kubectl delete pvc pvc-manual ; kubectl delete pv pv-manual
```

## Stretch
- Set the PV reclaim policy to `Retain`, delete the PVC, and observe the PV go to `Released`.
