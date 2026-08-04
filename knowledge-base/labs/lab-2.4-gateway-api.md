# Lab 2.4 — Gateway API

**Lesson:** [[2.4-gateway-api]] · **Cluster:** single node OK
**Goal:** install Gateway API CRDs + a controller, and route with a Gateway + HTTPRoute.

> Gateway API is not built in. This lab installs CRDs and the nginx-gateway-fabric (or ingress-nginx
> gateway) implementation. If install is flaky offline, do Parts A/D conceptually and focus on writing
> the HTTPRoute/Gateway YAML correctly — that's what the exam tests.

## Part A — Install CRDs + a controller (student)
```bash
# Gateway API CRDs (standard channel):
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.2.0/standard-install.yaml
kubectl get gatewayclass       # empty until a controller provides one
# Install an implementation that provides a GatewayClass, e.g. via its docs/Helm.
kubectl get crd | grep gateway.networking.k8s.io    # confirm CRDs exist
```

## Part B — Backends
```bash
kubectl create deploy web --image=nginx
kubectl expose deploy web --port=80
kubectl create deploy api --image=nginx
kubectl expose deploy api --port=80
```

## Part C — Gateway + HTTPRoute (the exam-relevant part)
```bash
kubectl apply -f - <<'EOF'
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata: {name: main-gw}
spec:
  gatewayClassName: <installed-class>      # from `kubectl get gatewayclass`
  listeners:
    - {name: http, protocol: HTTP, port: 80}
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata: {name: web-route}
spec:
  parentRefs: [{name: main-gw}]
  hostnames: ["shop.example.com"]
  rules:
    - matches: [{path: {type: PathPrefix, value: /api}}]
      backendRefs: [{name: api, port: 80}]
    - matches: [{path: {type: PathPrefix, value: /}}]
      backendRefs: [{name: web, port: 80}]
EOF
kubectl get gateway main-gw
kubectl get httproute web-route -o wide
kubectl describe httproute web-route        # check Accepted/ResolvedRefs conditions
```

## Part D — Test
```bash
# On macOS the node IP isn't routable — port-forward the gateway/controller service instead:
kubectl port-forward svc/<gateway-service> 8080:80 &      # service name is implementation-specific
curl --resolve shop.example.com:8080:127.0.0.1 http://shop.example.com:8080/api
kill %1
```

## Verification
- `kubectl get gatewayclass/gateway/httproute` all present; HTTPRoute conditions Accepted=True.
- Student can name the three resources and how `parentRefs`/`gatewayClassName` link them.

## Cleanup
```bash
kubectl delete httproute web-route
kubectl delete gateway main-gw
kubectl delete deploy web api ; kubectl delete svc web api
```

## Stretch
- Add a second `matches` on an HTTP header and split traffic with weighted `backendRefs`.
