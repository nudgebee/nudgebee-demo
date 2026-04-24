# 01 – Image Pull BackOff

A Deployment references an image that does not exist in any registry. The kubelet tries to pull it, fails with HTTP 403, and retries with exponential backoff.

## Apply

```bash
kubectl apply -f scenarios/01-image-pull-backoff/manifest.yaml
```

## Verify the failure

```bash
kubectl -n nudgebee-demo get pods -l nudgebee.io/demo-scenario=image-pull-backoff
```

The pod enters `ImagePullBackOff` within ~10 seconds.

## What Nudgebee should show

- **Event priority:** HIGH
- **Time to event:** ~2–3 minutes after apply
- **Reporter:** `image_pull_backoff_reporter`
- **Title:** `Failed to pull at least one image in pod demo-image-pull-backoff-<hash> in namespace nudgebee-demo`

### Evidence collected

- The full image reference that failed
- Pod events timeline (Pulling → Failed → BackOff)
- Pod details (node, image, resources, container status)
- The Deployment spec diff

### Expected RCA highlights

The RCA should identify:

- The pod is stuck in `ImagePullBackOff`
- The specific image that could not be pulled
- Likely causes: image does not exist, wrong tag, missing `imagePullSecrets`, or registry authentication failure
- Recommendations to verify the image reference and credentials

## Cleanup

```bash
kubectl delete -f scenarios/01-image-pull-backoff/manifest.yaml
```
