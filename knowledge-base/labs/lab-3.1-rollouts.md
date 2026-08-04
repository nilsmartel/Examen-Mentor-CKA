# Lab 3.1 — Rollouts & Rollbacks

**Lesson:** [[3.1-deployments-rollouts]] · **Cluster:** single node OK
**Goal:** roll a deployment forward, watch it, break it, and roll back.

## Part A — Rolling update (student)
```bash
kubectl create deploy web --image=nginx:1.26 --replicas=4
kubectl rollout status deploy/web
kubectl annotate deploy/web kubernetes.io/change-cause="init 1.26"
kubectl set image deploy/web nginx=nginx:1.27
kubectl annotate deploy/web kubernetes.io/change-cause="to 1.27" --overwrite
kubectl rollout status deploy/web
kubectl get rs                                   # old RS scaled to 0, new RS to 4
kubectl rollout history deploy/web
```

## Part B — Break the rollout (bad image) then recover
```bash
kubectl set image deploy/web nginx=nginx:doesnotexist999
kubectl rollout status deploy/web --timeout=30s     # will NOT complete
kubectl get pods                                     # new pods ImagePullBackOff
kubectl rollout undo deploy/web                      # revert to the working revision
kubectl rollout status deploy/web                    # healthy again
```
Ask: "How did you know the rollout was stuck, and what fixed it fastest?" (status hung + bad pods;
`rollout undo`.)

## Part C — Strategy knobs & restart
```bash
kubectl patch deploy web -p '{"spec":{"strategy":{"rollingUpdate":{"maxSurge":2,"maxUnavailable":0}}}}'
kubectl set image deploy/web nginx=nginx:1.27-alpine
kubectl get pods -w         # observe surge behavior (Ctrl-C when done)
kubectl rollout restart deploy/web                   # re-roll without a spec change
```

## Verification
- Student performs update → stuck update → `rollout undo` → healthy.
- Student can read `rollout history` and explain surge/unavailable.

## Cleanup
```bash
kubectl delete deploy web
```

## Stretch
- `kubectl rollout undo deploy/web --to-revision=1` and confirm the image reverted.
