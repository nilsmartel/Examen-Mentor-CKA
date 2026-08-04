# Lab 4.3 — StorageClasses & Dynamic Provisioning

**Lesson:** [[4.3-storage-classes-dynamic-provisioning]] · **Cluster:** single node OK
**Goal:** watch a PVC auto-provision a PV via the default StorageClass; see WaitForFirstConsumer.

## Part A — Inspect the default class (student)
```bash
kubectl get storageclass                         # note the one marked (default) — 'standard' on minikube
kubectl get sc standard -o yaml | grep -E "provisioner|volumeBindingMode|reclaimPolicy"
```

## Part B — Dynamic provisioning (no manual PV!)
```bash
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: PersistentVolumeClaim
metadata: {name: dyn}
spec:
  accessModes: [ReadWriteOnce]
  resources: {requests: {storage: 100Mi}}
  # no storageClassName -> uses the DEFAULT class
EOF
kubectl get pvc dyn                               # may be Pending (if WaitForFirstConsumer) or Bound
kubectl get pv                                    # a PV auto-created (if Immediate binding)
```

## Part C — WaitForFirstConsumer behavior
If the class is `WaitForFirstConsumer`, the PVC stays `Pending` until a pod consumes it:
```bash
kubectl run consumer --image=busybox --overrides='{"spec":{"volumes":[{"name":"d","persistentVolumeClaim":{"claimName":"dyn"}}],"containers":[{"name":"c","image":"busybox","command":["sh","-c","echo ok > /data/f; sleep 3600"],"volumeMounts":[{"name":"d","mountPath":"/data"}]}]}}'
kubectl get pvc dyn                               # NOW Bound (a pod consumed it)
kubectl get pv                                    # the dynamically created PV
kubectl exec consumer -- cat /data/f              # ok
```
Ask: "The PVC was Pending before the pod — bug or expected?" (Expected: WaitForFirstConsumer.)

## Part D — Make a custom StorageClass (optional)
```bash
kubectl apply -f - <<'EOF'
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata: {name: fast}
provisioner: k8s.io/minikube-hostpath
reclaimPolicy: Delete
volumeBindingMode: Immediate
EOF
kubectl get sc fast
```

## Verification
- A PV was created automatically for the PVC; student explains provisioner + binding mode.

## Cleanup
```bash
kubectl delete pod consumer --ignore-not-found
kubectl delete pvc dyn --ignore-not-found
kubectl delete sc fast --ignore-not-found
```

## Stretch
- Change which class is default by editing the `storageclass.kubernetes.io/is-default-class` annotation.
