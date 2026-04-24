# nudgebee-demo

Kubernetes failure scenarios you can apply to a cluster to see [Nudgebee](https://www.nudgebee.com)'s event detection, evidence collection, and AI-generated root cause analysis in action.

Each scenario produces a well-known, deterministic failure (e.g. `ImagePullBackOff`, `OOMKilled`, `CrashLoopBackOff`) so the resulting event and RCA are predictable and easy to reason about.

## Prerequisites

- A Kubernetes cluster (any distribution — GKE, EKS, AKS, k3s, kind, minikube).
- The Nudgebee Kubernetes collector installed and the cluster onboarded to your Nudgebee account. See [docs.nudgebee.com](https://docs.nudgebee.com/).
- `kubectl` configured against the target cluster.

## Quick start

```bash
# Create the demo namespace
kubectl apply -f namespace.yaml

# Apply any single scenario
kubectl apply -f scenarios/01-image-pull-backoff/manifest.yaml

# Or apply all scenarios at once
kubectl apply -f namespace.yaml
for d in scenarios/*/; do kubectl apply -f "$d/manifest.yaml"; done
```

Once a scenario is applied, open the Nudgebee UI and filter events by namespace `nudgebee-demo`. Expected wait times and what to look for are documented in each scenario's `README.md`.

## Scenarios

| # | Scenario | Failure | Time to event | Event priority |
|---|----------|---------|---------------|-----------------|
| 1 | [Image Pull BackOff](scenarios/01-image-pull-backoff/) | Container image cannot be pulled | ~2–3 min | HIGH |
| 2 | [CrashLoopBackOff](scenarios/02-crashloopbackoff/) | Container exits with non-zero code | ~2–3 min | HIGH |
| 3 | [OOMKilled](scenarios/03-oom-killed/) | Container killed for exceeding memory limit | ~2–3 min | HIGH |
| 4 | [Readiness probe failure](scenarios/04-readiness-probe-fail/) | Readiness probe keeps failing (HTTP 404) | immediate | DEBUG |
| 5 | [PVC Pending](scenarios/05-pvc-pending/) | PersistentVolumeClaim cannot bind | ~15 min | MEDIUM |

Scenarios 1–3 produce HIGH-priority events within a few minutes via dedicated Kubernetes reporters. Scenario 5 relies on a Prometheus alert (`KubePodNotReady`) which fires after ~15 minutes. Scenario 4 only produces a lower-priority Kubernetes warning event — see its README for details.

## Cleanup

```bash
# Remove a single scenario
kubectl delete -f scenarios/01-image-pull-backoff/manifest.yaml

# Remove everything including the namespace
kubectl delete namespace nudgebee-demo
```

## Labels

Every workload in this repo carries the following labels for easy filtering:

- `app.kubernetes.io/part-of: nudgebee-demo`
- `nudgebee.io/demo-scenario: <scenario-name>`

## License

MIT
