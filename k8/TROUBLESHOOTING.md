# Pod Deployment Troubleshooting Guide

## Quick Diagnostics

Run these commands in your EKS cluster to diagnose the issue:

### 1. Check if namespace exists
```bash
kubectl get namespace maintenance
```

### 2. Check deployment status
```bash
kubectl -n maintenance get deployment maintenance-page
kubectl -n maintenance describe deployment maintenance-page
```

### 3. Check pods
```bash
kubectl -n maintenance get pods
kubectl -n maintenance get pods -o wide
```

### 4. Check pod events and logs
```bash
# Get pod name first
POD_NAME=$(kubectl -n maintenance get pods -l app=maintenance-page -o jsonpath='{.items[0].metadata.name}')

# Check events
kubectl -n maintenance describe pod $POD_NAME

# Check logs
kubectl -n maintenance logs $POD_NAME
```

### 5. Check service endpoints
```bash
kubectl -n maintenance get svc maintenance-service
kubectl -n maintenance get endpoints maintenance-service
```

### 6. Check ingress
```bash
kubectl -n maintenance get ingress maintenance-ingress
kubectl -n maintenance describe ingress maintenance-ingress
```

### 7. Validate manifests
```bash
kubectl apply -k k8/ --dry-run=client -o yaml
```

## Common Issues & Solutions

### Issue: ImagePullBackOff
**Cause:** ECR image doesn't exist or authentication failed
**Solution:**
- Ensure CodeBuild successfully pushed the image: `aws ecr describe-images --repository-name maintenance-page`
- Check node IAM role has ECR access: `AmazonEC2ContainerRegistryReadOnly`

### Issue: CrashLoopBackOff
**Cause:** Container is exiting immediately (health check failing or nginx not starting)
**Solution:**
- Check logs: `kubectl -n maintenance logs <pod-name>`
- Verify /health endpoint is working: `curl http://localhost/health` (from inside pod)

### Issue: Pending (Pod not being scheduled)
**Cause:** Insufficient resources or node affinity issues
**Solution:**
- Check node resources: `kubectl top nodes`
- Check pod resource requests don't exceed node capacity

### Issue: Service has no endpoints
**Cause:** Service selector doesn't match pod labels
**Solution:**
- Already fixed: Service now uses `app: maintenance-page`

## Deploy Command

```bash
# From the repo root:
kubectl apply -k k8/

# Watch deployment
kubectl -n maintenance rollout status deployment/maintenance-page --timeout=300s

# Or manually check:
watch kubectl -n maintenance get pods
```

## Reset Everything

If you need to start fresh:
```bash
kubectl delete namespace maintenance
kubectl apply -k k8/
```
