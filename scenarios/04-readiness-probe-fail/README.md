# 04 – Readiness probe failure

A Deployment runs nginx with a readiness probe pointed at `/this-endpoint-does-not-exist`. The container starts and stays up, but every readiness check returns `HTTP 404`, so the Pod never becomes `Ready` and is not added to its Service's endpoints.

## Apply

```bash
kubectl apply -f scenarios/04-readiness-probe-fail/manifest.yaml
```

## Verify the failure

```bash
kubectl -n nudgebee-demo get pods -l nudgebee.io/demo-scenario=readiness-probe-fail
```

The pod is `Running` but `0/1` ready. You should see a `Warning Unhealthy` event:

```bash
kubectl -n nudgebee-demo describe pod -l nudgebee.io/demo-scenario=readiness-probe-fail | grep -A 1 Unhealthy
```

## What Nudgebee should show

- **Event priority:** DEBUG
- **Time to event:** immediate
- **Source:** `kubernetes_api_server`
- **Title:** `Unhealthy Warning for Pod nudgebee-demo/demo-readiness-probe-fail-<hash>`

### Evidence collected

- Pod events timeline (including `Readiness probe failed: HTTP probe failed with statuscode: 404`)
- Pod details
- Container logs (nginx access logs recording the 404)
- The Deployment spec diff

### Why this is DEBUG, not HIGH

Unlike `ImagePullBackOff`, `CrashLoopBackOff` and `OOMKilled`, readiness probe failures do not have a dedicated Nudgebee reporter that escalates them to HIGH. The standard Prometheus alert `KubePodNotReady` also does not fire for this pod because the pod's phase is `Running` (the container is up — it just does not pass the readiness check).

This scenario is useful for showing how Nudgebee ingests and enriches routine Kubernetes warning events, and for triggering an RCA manually from the Nudgebee UI on a lower-priority event.

### Expected RCA highlights

When the RCA is triggered (e.g. via **Ask Nudgebee** in the UI) it should identify:

- The pod is `Running` but not `Ready`
- The readiness probe is returning HTTP 404 on `/this-endpoint-does-not-exist`
- Recommendations to fix the probe path, expose the expected endpoint, or adjust probe configuration (initial delay, timeout, failure threshold)

## Cleanup

```bash
kubectl delete -f scenarios/04-readiness-probe-fail/manifest.yaml
```
