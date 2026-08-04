# kubectl Quick Reference (exam speed)

Mentor reference for the imperative habits that win the timed exam. Teach these relentlessly.

## Shell setup the student should do at the start of every exam

```bash
alias k=kubectl
export do="--dry-run=client -o yaml"   # "diff out" / dry-run yaml
export now="--force --grace-period=0"  # delete now
source <(kubectl completion bash)      # tab-completion
complete -o default -F __start_kubectl k
```
Then: `k run nginx --image=nginx $do > pod.yaml`

Set a default namespace to avoid repeating `-n`:
```bash
kubectl config set-context --current --namespace=<ns>
```

## Generate YAML fast (never write from scratch)

```bash
k run nginx --image=nginx $do > pod.yaml
k create deploy web --image=nginx --replicas=3 $do > deploy.yaml
k expose deploy web --port=80 --target-port=8080 $do > svc.yaml
k create svc clusterip web --tcp=80:8080 $do
k create cm app --from-literal=KEY=val $do
k create secret generic app --from-literal=pw=s3cret $do
k create job hello --image=busybox $do -- echo hi
k create cronjob hi --image=busybox --schedule="*/1 * * * *" $do -- date
k create role r --verb=get,list --resource=pods $do
k create rolebinding rb --role=r --user=jane $do
k create sa build
k create ingress ing --rule="host/path=svc:port" $do
k create quota q --hard=cpu=1,memory=1G,pods=2 $do
```

## Inspect & discover fields (allowed instead of memorizing)

```bash
k explain pod.spec.containers            # field discovery — use instead of guessing
k explain deploy.spec.strategy --recursive
k api-resources                          # kinds + short names + apiVersion
k get pod x -o yaml                       # see the full live object
k get pod x -o jsonpath='{.status.podIP}{"\n"}'
k get pods -o wide                        # node, IP
k get events --sort-by=.lastTimestamp -A  # what just happened
```

## Everyday verbs

```bash
k describe pod x                         # events + state — first stop when debugging
k logs x [-c container] [--previous] [-f]
k exec -it x -- sh
k get all -n ns
k rollout status/history/undo deploy/x
k scale deploy/x --replicas=5
k label/annotate ...
k top nodes ; k top pods                 # needs metrics-server
k delete pod x $now
```

## JSONPath / custom columns (common exam asks)

```bash
k get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}'
k get pods -o custom-columns=NAME:.metadata.name,NODE:.spec.nodeName
k get pods --sort-by=.metadata.creationTimestamp
k config view -o jsonpath='{.contexts[*].name}'   # list contexts
```

## Contexts (do this FIRST on every task)

```bash
k config get-contexts
k config use-context <cluster>
k config current-context
```

## Allowed documentation in the exam (the ONLY tab you get)

- `https://kubernetes.io/docs/` — main docs (copy YAML snippets from here)
- `https://kubernetes.io/blog/`
- `https://helm.sh/docs/`

Teach the student to navigate these fast. Useful landing pages:
- kubectl Cheat Sheet: `kubernetes.io/docs/reference/kubectl/cheatsheet/`
- Well-known bookmarks: PV/PVC, Ingress, NetworkPolicy, RBAC, and the Gateway API pages —
  the student should be able to reach each in seconds.
