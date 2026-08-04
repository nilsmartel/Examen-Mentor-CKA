# Lab 5.3 — Resource Monitoring (metrics-server + top)

**Lesson:** [[5.3-resource-usage-monitoring]] · **Cluster:** single node OK
**Goal:** enable metrics-server, use `kubectl top`, and identify the biggest resource consumer.

## Part A — Prove top doesn't work yet (student)
```bash
kubectl top nodes            # expect: "Metrics API not available"
```
Ask: "Why does this fail out of the box?" (metrics-server not installed.)

## Part B — Enable it
```bash
minikube addons enable metrics-server
kubectl get pods -n kube-system | grep metrics-server   # wait for Running
# wait ~60s for the first scrape window
kubectl top nodes
kubectl top pods -A
```

## Part C — Create load and find the hog (student)
```bash
kubectl create ns load
kubectl -n load run cpuhog --image=busybox -- sh -c "while true; do :; done"
kubectl -n load run idle --image=busybox -- sh -c "sleep 100000"
sleep 60
kubectl top pods -n load --sort-by=cpu     # cpuhog should be on top
kubectl top pod cpuhog -n load --containers
```
Ask the student to identify the highest-CPU pod cluster-wide:
```bash
kubectl top pods -A --sort-by=cpu | head
```

## Part D — Requests vs actual (concept reinforcement)
```bash
kubectl -n load create deploy cpuhog2 --image=busybox -- sh -c "while true; do :; done"
kubectl -n load set resources deploy/cpuhog2 --requests=cpu=100m --limits=cpu=200m   # set works on controllers
kubectl describe node minikube | grep -A6 "Allocated resources"   # committed REQUESTS
kubectl top node minikube                                          # ACTUAL usage
```
Discuss the difference: `describe node` = reservations; `top` = live usage.

## Verification
- `kubectl top nodes` and `kubectl top pods` return numbers.
- Student can name the top consumer and explain requests-vs-usage.

## Cleanup
```bash
kubectl delete ns load
```

## Stretch
- Leave metrics-server enabled — you need it for the HPA lab ([[3.3-autoscaling-hpa]]).
