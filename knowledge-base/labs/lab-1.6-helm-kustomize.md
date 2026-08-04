# Lab 1.6 — Helm & Kustomize

**Lesson:** [[1.6-helm-kustomize]] · **Cluster:** single node OK
**Goal:** install/upgrade/rollback a Helm release; build & apply a Kustomize overlay.

## Part A — Helm lifecycle (student)
> Uses a locally-scaffolded chart so the lab works offline and doesn't depend on external repos
> (the public Bitnami catalog was retired in 2025). Public-repo syntax is shown at the end for practice.
```bash
helm create demo                                   # scaffold a chart (defaults to an nginx deployment)
helm show values ./demo | head -40                 # what can I tune?
helm install web ./demo
helm list
kubectl get deploy,svc                              # release resources are named web-demo
# upgrade:
helm upgrade web ./demo --set replicaCount=3
kubectl get deploy web-demo -o jsonpath='{.spec.replicas}{"\n"}'   # 3
helm history web
# rollback:
helm rollback web 1
kubectl get deploy web-demo -o jsonpath='{.spec.replicas}{"\n"}'   # back to 1
# render without installing:
helm template web ./demo | head -30
helm uninstall web
# public-repo syntax (chart availability varies — practice the commands):
# helm repo add <name> <url> && helm repo update && helm search repo <term>
```
Ask: "How would you preview what a chart creates without touching the cluster?" (`helm template`.)

## Part B — Kustomize base + overlay (student builds files)
```bash
mkdir -p kdemo/base kdemo/overlays/prod
cat > kdemo/base/deployment.yaml <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata: {name: web}
spec:
  replicas: 1
  selector: {matchLabels: {app: web}}
  template:
    metadata: {labels: {app: web}}
    spec: {containers: [{name: web, image: nginx:1.27}]}
EOF
cat > kdemo/base/kustomization.yaml <<'EOF'
resources: [deployment.yaml]
EOF
cat > kdemo/overlays/prod/kustomization.yaml <<'EOF'
resources: [../../base]
namePrefix: prod-
labels:
  - pairs: {env: prod}
    includeSelectors: true
replicas:
  - {name: web, count: 3}
images:
  - {name: nginx, newTag: "1.27-alpine"}
EOF
# render, then apply:
kubectl kustomize kdemo/overlays/prod          # inspect the composed YAML
kubectl apply -k kdemo/overlays/prod
kubectl get deploy prod-web -o wide            # 3 replicas, image nginx:1.27-alpine, label env=prod
```

## Verification
- Helm: install→upgrade→rollback all reflected in `kubectl get`.
- Kustomize: `prod-web` exists with prefix, 3 replicas, overridden image, and `env=prod` label.

## Cleanup
```bash
kubectl delete -k kdemo/overlays/prod
rm -rf kdemo demo
helm uninstall web 2>/dev/null || true
```

## Stretch
- Add a `configMapGenerator` to the overlay and observe the hash-suffixed ConfigMap name.
