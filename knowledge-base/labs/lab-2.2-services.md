# Lab 2.2 — Services & Endpoints

**Lesson:** [[2.2-services-endpoints]] · **Cluster:** single node OK
**Goal:** create ClusterIP + NodePort, inspect endpoints, and break/fix a selector.

## Part A — ClusterIP from a deployment (student)
```bash
kubectl create deploy web --image=nginx --replicas=2
kubectl expose deploy web --port=80 --target-port=80          # ClusterIP
kubectl get svc web
kubectl get endpoints web                                     # 2 pod IPs (populated!)
kubectl run t --image=busybox -it --rm -- wget -qO- --timeout=3 http://web.default.svc.cluster.local
```

## Part B — Break the selector, watch endpoints empty (break-it/fix-it)
```bash
kubectl patch svc web -p '{"spec":{"selector":{"app":"nope"}}}'
kubectl get endpoints web                                     # EMPTY
kubectl run t --image=busybox -it --rm -- wget -qO- --timeout=3 http://web   # fails
# fix:
kubectl patch svc web -p '{"spec":{"selector":{"app":"web"}}}'
kubectl get endpoints web                                     # populated again
```
Ask: "What single command diagnosed this fastest?" (`kubectl get endpoints`.)

## Part C — NodePort
```bash
kubectl expose deploy web --name=web-np --type=NodePort --port=80
kubectl get svc web-np                                        # note the 3xxxx nodePort
minikube service web-np --url                                 # a reachable URL
curl $(minikube service web-np --url)
```

## Part D — targetPort mismatch (break-it/fix-it)
```bash
kubectl patch svc web -p '{"spec":{"ports":[{"port":80,"targetPort":8080}]}}'
kubectl get endpoints web                                     # shows :8080 — wrong
kubectl run t --image=busybox -it --rm -- wget -qO- --timeout=3 http://web   # refused
kubectl patch svc web -p '{"spec":{"ports":[{"port":80,"targetPort":80}]}}'  # fix
```

## Verification
- Student can populate/empty endpoints at will and explain why.
- Student distinguishes port / targetPort / nodePort.

## Cleanup
```bash
kubectl delete deploy web
kubectl delete svc web web-np
```

## Stretch
- Create a headless service (`--cluster-ip=None`) and `nslookup` it to see pod IPs directly.
