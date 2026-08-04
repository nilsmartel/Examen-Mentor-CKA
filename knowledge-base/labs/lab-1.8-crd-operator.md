# Lab 1.8 — CRDs & Operators

**Lesson:** [[1.8-crds-operators]] · **Cluster:** single node OK
**Goal:** install a CRD, create custom resources, and see that a CRD alone does nothing without a controller.

## Part A — Define a CRD (student)
```bash
kubectl apply -f - <<'EOF'
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: widgets.stable.example.com
spec:
  group: stable.example.com
  scope: Namespaced
  names: {plural: widgets, singular: widget, kind: Widget, shortNames: [wg]}
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                size: {type: integer}
EOF
kubectl get crd widgets.stable.example.com
kubectl api-resources | grep widgets       # your new kind now exists
kubectl explain widget.spec                 # works now that the CRD is installed
```

## Part B — Create custom resources
```bash
kubectl apply -f - <<'EOF'
apiVersion: stable.example.com/v1
kind: Widget
metadata: {name: w1}
spec: {size: 3}
EOF
kubectl get widgets
kubectl get wg w1 -o yaml                    # stored in the API — but nothing ACTS on it
```
Ask: "You set size: 3. Did anything happen in the cluster?" (No — there's no controller. A CRD only
stores data. An **operator** would reconcile it.)

## Part C — What an operator adds (conceptual + inspect a real one, optional)
Explain: an operator = these CRDs **+** a controller Deployment **+** RBAC. Optionally install a small
real operator via Helm to see the shape:
```bash
# example shape only (network permitting):
# helm install my-op <repo>/<operator>   -> creates CRDs + a controller pod + RBAC
kubectl get crds                            # an operator would add its own CRDs here
```

## Verification
- Student created a CRD, listed it via `api-resources`, and created CRs.
- Student can articulate: CRD = new type/storage; operator = CRD + reconciling controller + RBAC.

## Cleanup
```bash
kubectl delete widget w1
kubectl delete crd widgets.stable.example.com
```

## Stretch
- Add a `shortNames`/`printerColumns` field and re-apply; see `kubectl get wg` show your column.
