# Lab 3.4 — Self-Healing Primitives

**Lesson:** [[3.4-self-healing-primitives]] · **Cluster:** `--nodes 2` nice for DaemonSet, single node OK
**Goal:** see self-healing, and the liveness-vs-readiness behavior difference.

## Part A — Self-healing (student)
```bash
kubectl create deploy web --image=nginx --replicas=3
kubectl get pods -l app=web
kubectl delete pod <one-web-pod>            # controller recreates it immediately
kubectl get pods -l app=web -w              # a replacement appears
```

## Part B — Readiness probe removes from endpoints (NOT restart)
```bash
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: Pod
metadata: {name: readytest, labels: {app: rt}}
spec:
  containers:
    - name: c
      image: nginx
      readinessProbe:
        exec: {command: ["cat","/tmp/ready"]}   # file doesn't exist -> NOT ready
        periodSeconds: 3
EOF
kubectl expose pod readytest --port=80 --selector=app=rt
kubectl get pod readytest                     # READY 0/1, but STILL Running (not restarted)
kubectl get endpoints readytest               # EMPTY (excluded until ready)
kubectl exec readytest -- touch /tmp/ready    # make it ready
kubectl get pod readytest                     # READY 1/1
kubectl get endpoints readytest               # now populated
```
Ask: "Readiness failed — was the container restarted?" (No — that's liveness. Readiness only gates traffic.)

## Part C — Liveness probe restarts
```bash
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: Pod
metadata: {name: livetest}
spec:
  containers:
    - name: c
      image: busybox
      args: ["/bin/sh","-c","touch /tmp/alive; sleep 30; rm /tmp/alive; sleep 600"]
      livenessProbe:
        exec: {command: ["cat","/tmp/alive"]}
        periodSeconds: 5
EOF
kubectl get pod livetest -w                    # after ~30s the probe fails -> RESTARTS (count climbs)
```

## Part D — DaemonSet (if 2 nodes)
```bash
kubectl apply -f - <<'EOF'
apiVersion: apps/v1
kind: DaemonSet
metadata: {name: agent}
spec:
  selector: {matchLabels: {app: agent}}
  template:
    metadata: {labels: {app: agent}}
    spec: {containers: [{name: c, image: busybox, command: ["sleep","3600"]}]}
EOF
kubectl get pods -l app=agent -o wide          # one per node
```

## Verification
- Student can state: liveness ⇒ restart; readiness ⇒ endpoint gating; controllers recreate deleted pods.

## Cleanup
```bash
kubectl delete deploy web ; kubectl delete pod readytest livetest --ignore-not-found
kubectl delete svc readytest --ignore-not-found ; kubectl delete ds agent --ignore-not-found
```

## Stretch
- Build a StatefulSet + headless Service and observe `web-0`, `web-1` stable names/DNS.
