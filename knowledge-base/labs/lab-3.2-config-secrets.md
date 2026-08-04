# Lab 3.2 — ConfigMaps & Secrets

**Lesson:** [[3.2-configmaps-secrets]] · **Cluster:** single node OK
**Goal:** inject config as env vars and as mounted files; see the update-behavior difference.

## Part A — Create (student)
```bash
kubectl create ns cfg
kubectl -n cfg create configmap app --from-literal=MODE=prod --from-literal=COLOR=blue
kubectl -n cfg create secret generic db --from-literal=PASSWORD=s3cret
kubectl -n cfg get cm app -o yaml
kubectl -n cfg get secret db -o jsonpath='{.data.PASSWORD}' | base64 -d ; echo   # base64 != encryption
```

## Part B — As env vars
```bash
kubectl -n cfg run envpod --image=busybox --restart=Never \
  --overrides='{"spec":{"containers":[{"name":"c","image":"busybox","command":["env"],
    "envFrom":[{"configMapRef":{"name":"app"}}],
    "env":[{"name":"PASSWORD","valueFrom":{"secretKeyRef":{"name":"db","key":"PASSWORD"}}}]}]}}'
kubectl -n cfg logs envpod | grep -E "MODE|COLOR|PASSWORD"
```

## Part C — As mounted files
```bash
kubectl -n cfg apply -f - <<'EOF'
apiVersion: v1
kind: Pod
metadata: {name: filepod, namespace: cfg}
spec:
  containers:
    - name: c
      image: busybox
      command: ["sh","-c","sleep 3600"]
      volumeMounts:
        - {name: cfg, mountPath: /etc/appcfg, readOnly: true}
  volumes:
    - name: cfg
      configMap: {name: app}
EOF
kubectl -n cfg exec filepod -- ls /etc/appcfg          # MODE  COLOR (each key = a file)
kubectl -n cfg exec filepod -- cat /etc/appcfg/COLOR   # blue
```

## Part D — Update behavior (the exam trap)
```bash
kubectl -n cfg patch configmap app -p '{"data":{"COLOR":"red"}}'
kubectl -n cfg exec filepod -- cat /etc/appcfg/COLOR   # eventually 'red' (mounted files update)
# env-var consumers do NOT change until restart:
kubectl -n cfg logs envpod | grep COLOR                # still blue -> needs a restart
```

## Verification
- Student consumed config both ways and can explain: env captured at start; mounts update live.

## Cleanup
```bash
kubectl delete ns cfg
```

## Stretch
- `kubectl create secret tls` from a self-signed cert/key and mount it.
