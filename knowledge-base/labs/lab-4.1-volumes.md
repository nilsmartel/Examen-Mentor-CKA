# Lab 4.1 — Volumes (ephemeral vs persistent)

**Lesson:** [[4.1-volumes-access-modes-reclaim]] · **Cluster:** single node OK
**Goal:** feel the difference between emptyDir (dies with pod) and PV-backed storage (survives).

## Part A — emptyDir shared between containers (student)
```bash
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: Pod
metadata: {name: shared}
spec:
  containers:
    - name: writer
      image: busybox
      command: ["sh","-c","echo hello > /data/msg; sleep 3600"]
      volumeMounts: [{name: d, mountPath: /data}]
    - name: reader
      image: busybox
      command: ["sh","-c","sleep 3600"]
      volumeMounts: [{name: d, mountPath: /data}]
  volumes:
    - {name: d, emptyDir: {}}
EOF
kubectl exec shared -c reader -- cat /data/msg      # 'hello' — shared between containers
```

## Part B — emptyDir does NOT survive pod deletion
```bash
kubectl delete pod shared
# recreate (same manifest) and note /data/msg is only there because writer rewrites it —
# the previous emptyDir contents were destroyed with the pod.
```
Ask: "Where would you put data that must outlive the pod?" (a PersistentVolume — next lab.)

## Part C — hostPath persistence on the node
```bash
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: Pod
metadata: {name: hp}
spec:
  containers:
    - name: c
      image: busybox
      command: ["sh","-c","echo persisted > /data/f; sleep 3600"]
      volumeMounts: [{name: d, mountPath: /data}]
  volumes:
    - {name: d, hostPath: {path: /mnt/labdata, type: DirectoryOrCreate}}
EOF
kubectl exec hp -- cat /data/f
kubectl delete pod hp
# the file remains on the NODE at /mnt/labdata:
minikube ssh -- cat /mnt/labdata/f                  # 'persisted' — outlived the pod
```

## Verification
- Student can state which volume types survive pod deletion and why.

## Cleanup
```bash
kubectl delete pod shared hp --ignore-not-found
minikube ssh -- sudo rm -rf /mnt/labdata 2>/dev/null || true
```

## Stretch
- Add access-mode reasoning: which mode would a multi-node RW share need? (RWX.)
