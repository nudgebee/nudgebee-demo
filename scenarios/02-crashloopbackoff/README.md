# 02 – CrashLoopBackOff

A Deployment starts a container that immediately exits with a non-zero status and a clear fatal-error message written to stderr. The kubelet keeps restarting the container, which eventually enters `CrashLoopBackOff`.

## Apply

```bash
kubectl apply -f scenarios/02-crashloopbackoff/manifest.yaml
```

## Verify the failure

```bash
kubectl -n nudgebee-demo get pods -l nudgebee.io/demo-scenario=crashloopbackoff
kubectl -n nudgebee-demo logs -l nudgebee.io/demo-scenario=crashloopbackoff --tail=5
```

The pod reaches `CrashLoopBackOff` after a couple of restarts. Its logs include:

```
demo-crashloopbackoff: starting
demo-crashloopbackoff: fatal: required config 'DATABASE_URL' is missing
```

## What Nudgebee should show

- **Event priority:** HIGH
- **Time to event:** ~2–3 minutes after apply
- **Reporter:** `report_crash_loop`
- **Title:** `Crashing pod demo-crashloopbackoff-<hash> in namespace nudgebee-demo`

### Evidence collected

- Restart count, waiting reason (`CrashLoopBackOff`), termination reason (`Error`)
- Container logs (captures the fatal error message)
- Pod events and pod details
- The Deployment spec diff

### Expected RCA highlights

Because the stderr message is captured in the log evidence, the RCA should identify:

- The pod is in `CrashLoopBackOff`
- The specific error from logs: a missing required configuration `DATABASE_URL`
- Recommendations to verify that the expected environment variables / ConfigMap / Secret are wired up correctly

## Cleanup

```bash
kubectl delete -f scenarios/02-crashloopbackoff/manifest.yaml
```
