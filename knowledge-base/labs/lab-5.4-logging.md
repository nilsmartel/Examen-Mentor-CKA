# Lab 5.4 — Container Output & Logging

**Lesson:** [[5.4-container-output-logs]] · **Cluster:** single node OK
**Goal:** read logs (incl. `--previous`), and diagnose each failure state by its signature.

## Part A — A CrashLoopBackOff (student)
```bash
kubectl create ns dbg
kubectl -n dbg run crasher --image=busybox -- sh -c "echo starting; sleep 2; echo dying; exit 1"
sleep 20
kubectl -n dbg get pod crasher            # CrashLoopBackOff
kubectl -n dbg logs crasher               # may be mid-restart
kubectl -n dbg logs crasher --previous    # the crashed instance's output <-- key command
kubectl -n dbg describe pod crasher       # Last State: Terminated, Exit Code 1
```
Ask: "Which flag showed you the death message?" (`--previous`.)

## Part B — ImagePullBackOff (different root cause)
```bash
kubectl -n dbg run badimage --image=nginx:doesnotexist123
sleep 15
kubectl -n dbg get pod badimage           # ImagePullBackOff
kubectl -n dbg describe pod badimage      # Events: failed to pull, tag not found
```
Contrast with Part A: this never ran → not an app bug. Fix by correcting the tag:
```bash
kubectl -n dbg set image pod/badimage badimage=nginx:1.27 2>/dev/null || \
kubectl -n dbg delete pod badimage && kubectl -n dbg run badimage --image=nginx:1.27
```

## Part C — OOMKilled / exit 137
```bash
kubectl -n dbg run oom --image=polinux/stress \
  --overrides='{"spec":{"containers":[{"name":"oom","image":"polinux/stress","resources":{"limits":{"memory":"20Mi"}},"command":["stress","--vm","1","--vm-bytes","150M","--vm-hang","1"]}]}}'
sleep 20
kubectl -n dbg describe pod oom | grep -A3 "Last State"   # Reason: OOMKilled, Exit Code: 137
```
Ask: "What does 137 mean and how do you fix it?" (SIGKILL/OOM; raise the memory limit.)

## Part D — Pending (no logs to read — read events instead)
```bash
kubectl -n dbg run toobig --image=nginx \
  --overrides='{"spec":{"containers":[{"name":"toobig","image":"nginx","resources":{"requests":{"cpu":"100"}}}]}}'
sleep 5
kubectl -n dbg get pod toobig             # Pending
kubectl -n dbg describe pod toobig        # Events: Insufficient cpu
```
Point: Pending pods have no logs; the answer is in **Events**.

## Verification
- Student can map each state → the command that reveals its cause
  (CrashLoop→`logs --previous`, ImagePull→`describe`/Events, OOM→`describe` Last State, Pending→Events).

## Cleanup
```bash
kubectl delete ns dbg
```

## Stretch
- `kubectl debug -it <pod> --image=busybox --target=<container>` to attach an ephemeral debug container.
- `kubectl get events -n dbg --sort-by=.lastTimestamp`.
