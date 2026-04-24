# 05 – PVC Pending

A Deployment mounts a PersistentVolumeClaim that references a StorageClass which does not exist. The PVC stays `Pending` forever, which in turn keeps the Pod in `Pending` with `FailedScheduling` because the volume cannot be bound.

## Apply

```bash
kubectl apply -f scenarios/05-pvc-pending/manifest.yaml
```

## Verify the failure

```bash
kubectl -n nudgebee-demo get pvc,pod -l nudgebee.io/demo-scenario=pvc-pending
```

The PVC is `Pending` and the Pod is stuck `Pending` due to unbound volume.

## What Nudgebee should show

This scenario has two stages:

### Immediate (seconds)

- **Event priority:** DEBUG
- **Source:** `kubernetes_api_server`
- **Titles:**
  - `ProvisioningFailed Warning for PersistentVolumeClaim nudgebee-demo/demo-pvc-pending`
  - `FailedScheduling Warning for Pod nudgebee-demo/demo-pvc-pending-<hash>`

### After ~15 minutes (via Prometheus)

- **Event priority:** MEDIUM
- **Source:** `pagerduty_webhook`
- **Title:** `KubePodNotReady` (fires because the pod stays in `Pending` phase for ≥ 15 min)

### Evidence collected

- Pod events (`FailedScheduling`, unbound immediate PersistentVolumeClaims)
- Pod details
- The Deployment and PVC spec diffs
- Alert rule details and Prometheus query data (once the MEDIUM alert fires)

### Expected RCA highlights

When the RCA is triggered it should identify:

- The Pod cannot be scheduled because its PVC is unbound
- The PVC references a StorageClass (`does-not-exist`) that is not present in the cluster
- Recommendations to create the missing StorageClass, fix the claim to reference a valid one, or manually provision a PersistentVolume

## Cleanup

```bash
kubectl delete -f scenarios/05-pvc-pending/manifest.yaml
```
