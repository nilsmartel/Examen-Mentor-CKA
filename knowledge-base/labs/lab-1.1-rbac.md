# Lab 1.1 — RBAC

**Lesson:** [[1.1-rbac]] · **Cluster:** single node OK
**Goal:** grant a user and a ServiceAccount scoped permissions and verify with `auth can-i`.

## Part A — User, Role, RoleBinding (student, imperative)
```bash
kubectl create ns web
kubectl create role pod-reader --verb=get,list,watch --resource=pods -n web
kubectl create rolebinding jane-reader --role=pod-reader --user=jane -n web
# verify:
kubectl auth can-i list pods   -n web     --as=jane      # yes
kubectl auth can-i delete pods -n web     --as=jane      # no
kubectl auth can-i list pods   -n default --as=jane      # no (namespaced!)
```
Ask: "Why is the last one 'no'?" (Role/RoleBinding are namespace-scoped to `web`.)

## Part B — Deliberate wrong apiGroup (break-it/fix-it)
```bash
kubectl create role deploy-reader --verb=get,list --resource=pods -n web   # WRONG resource for the goal
kubectl create rolebinding jane-deploy --role=deploy-reader --user=jane -n web
kubectl auth can-i list deployments -n web --as=jane      # no — deployments are in apps group
# fix by editing the role to include deployments:
kubectl edit role deploy-reader -n web   # add apiGroups: ["apps"], resources: ["deployments"]
kubectl auth can-i list deployments -n web --as=jane      # yes
```
Reinforce: deployments live in the `apps` API group, not core `""`.

## Part C — ServiceAccount for a pod
```bash
kubectl create sa reader-sa -n web
kubectl create rolebinding sa-reader --role=pod-reader --serviceaccount=web:reader-sa -n web
kubectl auth can-i list pods -n web --as=system:serviceaccount:web:reader-sa   # yes
# run a pod as that SA and test from inside:
# any image containing the kubectl binary works (the public bitnami/kubectl image was retired in 2025):
kubectl -n web run test --image=bitnamilegacy/kubectl --overrides='{"spec":{"serviceAccountName":"reader-sa"}}' \
  --command -- sleep 3600
kubectl -n web exec test -- kubectl get pods -n web       # works
kubectl -n web exec test -- kubectl get pods -n default   # forbidden
```

## Part D — ClusterRole reused per-namespace
```bash
kubectl create clusterrole cm-reader --verb=get,list --resource=configmaps
kubectl create rolebinding cm-web --clusterrole=cm-reader --user=jane -n web   # RoleBinding -> ClusterRole
kubectl auth can-i list configmaps -n web     --as=jane   # yes
kubectl auth can-i list configmaps -n default --as=jane   # no (bound only in web)
```

## Verification
- `auth can-i` gives the expected yes/no for each subject and namespace.
- Student can explain Role vs ClusterRole and the RoleBinding→ClusterRole pattern.

## Cleanup
```bash
kubectl delete ns web
kubectl delete clusterrole cm-reader
```

## Stretch
- `kubectl auth can-i --list --as=jane -n web` to enumerate everything jane can do.
