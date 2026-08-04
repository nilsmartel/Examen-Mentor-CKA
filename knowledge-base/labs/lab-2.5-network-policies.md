# Lab 2.5 — Network Policies

**Lesson:** [[2.5-network-policies]] · **Cluster:** needs a policy-enforcing CNI
**Goal:** implement default-deny + a targeted allow, and prove enforcement.

> ⚠️ minikube's default CNI may NOT enforce NetworkPolicy. Start with Calico:
> `minikube delete && minikube start --driver=podman --cni=calico`
> If you can't, do the lab as YAML authoring practice and verify with `describe`.

## Part A — Baseline: everything talks (student)
```bash
kubectl create ns np
kubectl -n np run web --image=nginx --labels=app=web --port=80
kubectl -n np expose pod web --port=80
kubectl -n np run client --image=busybox --labels=role=frontend -- sleep 3600
kubectl -n np run stranger --image=busybox --labels=role=other -- sleep 3600
kubectl -n np exec client   -- wget -qO- --timeout=3 http://web   # works
kubectl -n np exec stranger -- wget -qO- --timeout=3 http://web   # works
```

## Part B — Default-deny ingress
```bash
kubectl -n np apply -f - <<'EOF'
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: {name: default-deny-ingress, namespace: np}
spec:
  podSelector: {}
  policyTypes: [Ingress]
EOF
kubectl -n np exec client -- wget -qO- --timeout=3 http://web   # now BLOCKED (times out)
```

## Part C — Allow only role=frontend on port 80
```bash
kubectl -n np apply -f - <<'EOF'
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: {name: allow-frontend, namespace: np}
spec:
  podSelector: {matchLabels: {app: web}}
  policyTypes: [Ingress]
  ingress:
    - from:
        - podSelector: {matchLabels: {role: frontend}}
      ports:
        - {protocol: TCP, port: 80}
EOF
kubectl -n np exec client   -- wget -qO- --timeout=3 http://web   # ALLOWED
kubectl -n np exec stranger -- wget -qO- --timeout=3 http://web   # still BLOCKED
```
Ask: "Rules are additive — how would you also allow role=admin?" (add a second `-` podSelector item.)

## Part D — Egress + DNS gotcha (discuss/optional)
Apply a default-deny egress and watch DNS break; then add an egress rule allowing UDP/TCP 53 to
kube-system. Reinforces the DNS-under-egress trap.

## Verification
- After Part C, only `role=frontend` reaches `web`; `stranger` is blocked.
- Student explains selected-pod default-deny and additive rules.

## Cleanup
```bash
kubectl delete ns np
```

## Stretch
- Change the allow to combine `namespaceSelector` + `podSelector` in one item (AND) vs two items (OR)
  and observe the difference.
