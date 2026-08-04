# Lab 5.5 — Services & Networking Troubleshooting

**Lesson:** [[5.5-services-networking-troubleshooting]] · **Cluster:** single node OK
**Goal:** fix a Service that returns nothing by walking the request path (selector → endpoints →
targetPort → DNS → NetworkPolicy).

## Setup — a deliberately broken Service (mentor seeds; or student applies)
```bash
kubectl create ns net
kubectl -n net create deploy web --image=nginx --port=80
kubectl -n net label deploy web app=web --overwrite
# Service with a WRONG selector on purpose:
kubectl -n net create svc clusterip web --tcp=80:80
kubectl -n net patch svc web -p '{"spec":{"selector":{"app":"frontend"}}}'
```

## Part A — Observe the failure (student)
```bash
kubectl -n net run tmp --image=busybox -it --rm -- sh
# inside:
wget -qO- --timeout=3 http://web        # hangs/fails
nslookup web                            # DNS RESOLVES (name exists) — so it's not DNS
exit
```

## Part B — Diagnose the fastest way (student)
```bash
kubectl -n net get endpoints web        # EMPTY -> selector/label mismatch!
kubectl -n net get svc web -o wide      # selector: app=frontend
kubectl -n net get pods --show-labels   # pods have app=web
```
Student concludes: selector `app=frontend` ≠ pod label `app=web`.

## Part C — Fix #1 (selector) and verify
```bash
kubectl -n net patch svc web -p '{"spec":{"selector":{"app":"web"}}}'
kubectl -n net get endpoints web        # now populated
kubectl -n net run tmp --image=busybox -it --rm -- wget -qO- --timeout=3 http://web  # nginx welcome
```

## Part D — Second break: wrong targetPort
```bash
kubectl -n net patch svc web -p '{"spec":{"ports":[{"port":80,"targetPort":8080}]}}'
kubectl -n net run tmp --image=busybox -it --rm -- wget -qO- --timeout=3 http://web  # refused
# endpoints still populated, so it's a PORT problem:
kubectl -n net get endpoints web        # shows :8080 (wrong)
kubectl -n net patch svc web -p '{"spec":{"ports":[{"port":80,"targetPort":80}]}}'  # fix
kubectl -n net run tmp --image=busybox -it --rm -- wget -qO- --timeout=3 http://web  # works
```

## Part E — Third break: default-deny NetworkPolicy (needs a CNI that enforces it)
> Note: minikube's default CNI may not enforce NetworkPolicy. If not, teach this conceptually or use
> `minikube start --cni=calico`. Flag this to the student honestly.
```bash
kubectl -n net apply -f - <<'EOF'
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: {name: default-deny, namespace: net}
spec: {podSelector: {}, policyTypes: [Ingress]}
EOF
# now traffic is blocked despite good endpoints & port:
kubectl -n net get netpol
kubectl -n net delete netpol default-deny   # remove to restore
```

## Verification
- Student can recite the checklist: endpoints? pods Ready? targetPort? DNS? NetworkPolicy?
- The Service serves nginx again.

## Cleanup
```bash
kubectl delete ns net
```

## Stretch
- Break it by making the pod's readiness probe fail and watch it drop out of endpoints.
