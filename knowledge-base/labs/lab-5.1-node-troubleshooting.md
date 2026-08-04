# Lab 5.1 — Node Troubleshooting (kubelet)

**Lesson:** [[5.1-cluster-node-troubleshooting]] · **Cluster:** single node OK (`minikube start --driver=podman`)
**Goal:** take a healthy node → break the kubelet → diagnose from `kubectl` and the node → restore.

> Mentor: have the STUDENT type every command. You seed the break, they diagnose & fix.

## Part A — Baseline (student)
```bash
kubectl get nodes                 # note it's Ready
kubectl describe node minikube    # scroll to Conditions + read each; note the kubelet version
kubectl get pods -A -o wide       # what's running where
```
Ask: "Which condition tells you the node is healthy, and what posts it?" (Ready; the kubelet.)

## Part B — Break it (mentor seeds, or guide student)
On the node, stop the kubelet:
```bash
minikube ssh
sudo systemctl stop kubelet
exit
```
Wait ~40–60s.

## Part C — Diagnose (student)
```bash
kubectl get nodes                 # should now be NotReady
kubectl describe node minikube    # Ready condition -> False/Unknown; read the message
```
Then go to the node and find the cause:
```bash
minikube ssh
sudo systemctl status kubelet     # inactive (dead)
sudo journalctl -u kubelet | tail -20
```
Ask the student to narrate what happened before fixing.

## Part D — Fix & verify (student)
```bash
# still on the node:
sudo systemctl start kubelet
sudo systemctl status kubelet     # active (running)
exit
kubectl get nodes                 # back to Ready within ~30s
```

## Part E — Break-it/fix-it curveball (config-level)
Introduce a bad kubelet config to simulate the harder exam variant:
```bash
minikube ssh
sudo cp /var/lib/kubelet/config.yaml /tmp/config.yaml.bak
echo "thisIsNotValid: [" | sudo tee -a /var/lib/kubelet/config.yaml   # corrupt it
sudo systemctl restart kubelet
sudo systemctl status kubelet     # failed
sudo journalctl -u kubelet | tail -20   # READ the parse error
# fix:
sudo cp /tmp/config.yaml.bak /var/lib/kubelet/config.yaml
sudo systemctl restart kubelet
exit
kubectl get nodes                 # Ready
```

## Verification
- `kubectl get nodes` → `Ready`.
- Student can state: NotReady = kubelet heartbeat lost; fix is on the node (systemd/config), not the API.

## Cleanup
Nothing to delete. Ensure node is `Ready`.

## Stretch
- `kubectl cordon minikube` then `kubectl uncordon minikube`; observe `SchedulingDisabled`.
- Cordon, create a pod, watch it stay `Pending`; uncordon and watch it schedule.
