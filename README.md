# CKA Study Mentor

An interactive, hands-on tutor that coaches you through the **Certified Kubernetes Administrator
(CKA)** exam — powered by Claude Code in this repository. You learn by *talking to the mentor and
doing labs* in a real local cluster; you don't read the material yourself.

## Prerequisites (macOS)

All four should already be installed. Verify:

| Tool | Verify command |
|------|----------------|
| podman | `podman version` |
| minikube | `minikube version` |
| kubectl | `kubectl version --client` |
| helm | `helm version` |

Bootstrap the practice cluster once:

```bash
podman machine start          # if not already running
minikube start --driver=podman
kubectl get nodes             # should show a Ready node
```

Some labs use a second node (`minikube start --driver=podman --nodes 2`) or addons
(`minikube addons enable metrics-server`) — the mentor tells you when and handles it with you.
More detail: [`reference/environment-setup.md`](reference/environment-setup.md).

## How to use it

1. Open this repo in **Claude Code**.
2. Say **"let's study"** (or "where are we?", or "let's continue").
3. The mentor (Kube) takes over: it reviews past topics, teaches the next one conversationally,
   runs a hands-on lab with you, quizzes you, and records your progress.

That's it. No setup beyond the cluster. The agent's behavior lives in
[`CLAUDE.md`](CLAUDE.md) and loads automatically.

## Tracking progress

Your progress lives in [`progress.md`](progress.md) — the mentor keeps it current. It shows you a
progress dashboard (weighted "~X% done", strong/weak domains, what's next) **at the start and end of
every session**, so you always see the number move. You can also ask **"how far are we?"** any time.

## What's in here

```
CLAUDE.md            The mentor agent's instructions (loaded automatically)
progress.md          Your live progress + % complete (mentor maintains it)
knowledge-base/      Curriculum map, 27 lessons across 5 domains, labs, recall bank
reference/           Environment setup, kubectl quick-ref, teaching methodology
```

You never need to open `knowledge-base/` yourself — the mentor teaches from it. It's there if
you're ever curious.

---
*Targets Kubernetes v1.35 · exam is performance-based, 2 hours, 66% to pass.*
