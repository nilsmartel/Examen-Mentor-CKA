
# Services & Endpoints (2.2)

```bash
kubectl expose deploy web --port=80 --target-port=80                    # ClusterIP over a deployment's pods
kubectl expose deploy web --name=web-np --type=NodePort --port=80       # NodePort (adds 3xxxx on every node)
kubectl get ep web                                                      # #1 debug: empty => selector/readiness; IPs w/ wrong port => targetPort
kubectl get endpointslices -l kubernetes.io/service-name=web            # slices named <svc>-<rand>, find by label not name
kubectl patch svc web -p '{"spec":{"selector":{"app":"web"}}}'         # fix a broken selector
kubectl api-resources | grep -i policy                                  # find a resource's short name / group fast
# port=client-facing svc port | targetPort=pod's real listen port (must match app) | nodePort=external 3xxxx
# headless: clusterIP:None => no VIP, DNS returns pod IPs; + StatefulSet => web-0.web.ns.svc.cluster.local
```

# Pod Connectivity & Network Model (2.1)

```bash
kubectl get pods -A -o wide                                              # pod IPs + which node each is on
kubectl run client --image=busybox -it --rm -- wget -qO- --timeout=3 http://<pod-ip>   # test pod-to-pod by IP
kubectl exec twocon -c shell -- wget -qO- http://localhost:80           # same pod = same netns, reach via localhost
kubectl logs <pod> -c <container>                                       # e.g. spot "Address already in use" port clash
# no CNI -> nodes NotReady; CNI broken -> pods ContainerCreating; fix = apply/repair CNI manifest
# pod CIDR e.g. 10.244.0.0/16 (real pod IPs) | service CIDR e.g. 10.96.0.0/12 (virtual, kube-proxy-translated)
```
