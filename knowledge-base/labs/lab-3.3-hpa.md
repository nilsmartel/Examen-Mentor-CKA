# Lab 3.3 — Horizontal Pod Autoscaler

**Lesson:** [[3.3-autoscaling-hpa]] · **Cluster:** single node OK · **Needs metrics-server**
**Goal:** create an HPA, generate load, and watch it scale up and back down.

## Part A — Prereqs (student)
```bash
minikube addons enable metrics-server        # if not already (see lab 5.3)
kubectl top nodes                            # confirm metrics work first
```

## Part B — Deploy WITH cpu requests (HPA needs them)
```bash
kubectl create deploy php --image=registry.k8s.io/hpa-example --port=80
kubectl set resources deploy php --requests=cpu=100m --limits=cpu=200m
kubectl expose deploy php --port=80
kubectl autoscale deploy php --cpu-percent=50 --min=1 --max=5
kubectl get hpa                              # TARGETS may briefly show <unknown>, then 0%/50%
```
Ask: "If TARGETS stays `<unknown>`, what two things do you check?" (metrics-server; cpu requests.)

## Part C — Generate load and watch scale-up
```bash
# in one terminal (student):
kubectl run load --image=busybox --restart=Never -it --rm -- \
  /bin/sh -c "while true; do wget -q -O- http://php; done"
# in another:
kubectl get hpa php -w         # watch current% climb past 50% and replicas grow toward 5
kubectl get pods -l app=php -w
```

## Part D — Stop load and watch scale-down (slow by design)
```bash
# Ctrl-C the load pod. Then:
kubectl get hpa php -w         # utilization drops; after the stabilization window replicas shrink to 1
```
Point: scale-down is intentionally slow (stabilization) to avoid flapping.

## Verification
- Replicas increase under load and decrease after — student can read the HPA math and prereqs.

## Cleanup
```bash
kubectl delete hpa php ; kubectl delete deploy php ; kubectl delete svc php
```

## Stretch
- Rewrite the HPA as `autoscaling/v2` YAML with a memory metric added.
