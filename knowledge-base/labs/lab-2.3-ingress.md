# Lab 2.3 — Ingress

**Lesson:** [[2.3-ingress-controllers]] · **Cluster:** single node OK
**Goal:** enable the controller, route two paths to two Services, and test via curl.

## Part A — Controller + two backends (student)
```bash
minikube addons enable ingress
kubectl get pods -n ingress-nginx                       # wait for controller Running

kubectl create deploy web --image=nginx --port=80
kubectl expose deploy web --port=80
kubectl create deploy api --image=hashicorp/http-echo -- -text=hello-api -listen=:5678
kubectl expose deploy api --port=80 --target-port=5678       # http-echo listens on 5678
```

## Part B — Ingress with host + paths
```bash
kubectl create ingress shop --class=nginx \
  --rule="shop.example.com/api*=api:80" \
  --rule="shop.example.com/*=web:80"
kubectl get ingress shop                                # wait for ADDRESS to populate
kubectl describe ingress shop
```

## Part C — Test (reliable on the macOS podman driver)
> On macOS the node IP (`minikube ip`) isn't routable from the host. Port-forward the ingress
> controller and curl `127.0.0.1` — works on any driver. (`--resolve` fakes the DNS name.)
```bash
kubectl port-forward -n ingress-nginx svc/ingress-nginx-controller 8080:80 &
curl --resolve shop.example.com:8080:127.0.0.1 http://shop.example.com:8080/       # -> web (nginx)
curl --resolve shop.example.com:8080:127.0.0.1 http://shop.example.com:8080/api    # -> hello-api
kill %1     # stop the port-forward when done
```

## Part D — Break-it/fix-it: missing controller / wrong backend
```bash
kubectl edit ingress shop        # point a rule at a non-existent service, re-test -> 503
# observe the failure, then fix the backend name and re-test.
```
Reinforce: an Ingress is inert without a controller, and a rule is only as good as the Service it targets.

## Verification
- Both paths route to the correct backend via curl.
- Student can explain controller-vs-resource and the role of `pathType`/`ingressClassName`.

## Cleanup
```bash
kubectl delete ingress shop
kubectl delete deploy web api --ignore-not-found
kubectl delete svc web api --ignore-not-found
kubectl delete pod api --ignore-not-found
```

## Stretch
- Add TLS: create a self-signed `kubernetes.io/tls` secret and a `tls:` block; curl with `-k https://`.
