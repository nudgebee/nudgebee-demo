# 03 – OOMKilled

A Deployment runs `stress` with a 256 MB memory allocation but the container's memory limit is 64 MB. The kubelet kills the container with `OOMKilled` and restarts it, which results in an OOM crash loop.

## Apply

```bash
kubectl apply -f scenarios/03-oom-killed/manifest.yaml
```

## Verify the failure

```bash
kubectl -n nudgebee-demo get pods -l nudgebee.io/demo-scenario=oom-killed
kubectl -n nudgebee-demo get pod -l nudgebee.io/demo-scenario=oom-killed -o jsonpath='{.items[0].status.containerStatuses[0].lastState.terminated.reason}'
```

The container is killed with `OOMKilled` within ~15 seconds and restarts.

## What Nudgebee should show

- **Event priority:** HIGH
- **Time to event:** ~2–3 minutes after apply
- **Reporter:** `pod_oom_killer_enricher`
- **Title:** `Pod demo-oom-killed-<hash> in namespace nudgebee-demo OOMKilled results`

### Evidence collected

- OOM container matrix (memory requests, limits, usage samples over time)
- Noisy-neighbours analysis (other pods on the same node)
- Node-level memory metrics
- Pod events and pod details
- Container logs
- The Deployment spec diff

### Expected RCA highlights

The RCA should identify:

- The container was killed because it exceeded the 64 Mi memory limit
- The actual memory usage pattern vs the configured limit
- Recommendations to raise the memory limit, reduce the workload's memory footprint, or investigate what is producing the memory growth

## Cleanup

```bash
kubectl delete -f scenarios/03-oom-killed/manifest.yaml
```
