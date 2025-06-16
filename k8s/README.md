# Price List Search - Kubernetes Deployment Guide

This guide explains how to deploy the Price List Search application to a Kubernetes cluster.

## Prerequisites

- Kubernetes cluster
- kubectl configured to access your cluster
- Docker registry access
- Domain name (for ingress configuration)

## Directory Structure

```
k8s/
├── deployment.yaml   # Deployment configuration
├── service.yaml      # Service configuration
├── ingress.yaml     # Ingress configuration
└── README.md        # This file
```

## Deployment Steps

### 1. Build and Push Docker Image

```bash
# Build the Docker image
docker build -t your-registry/price-list-search:latest .

# Push to your registry
docker push your-registry/price-list-search:latest
```

### 2. Update Configuration

1. Edit `deployment.yaml`:
   - Update image name to match your registry
   - Adjust resource limits if needed

2. Edit `ingress.yaml`:
   - Update host name to match your domain

### 3. Deploy to Kubernetes

```bash
# Create namespace
kubectl create namespace price-list

# Apply configurations
kubectl apply -f k8s/ -n price-list
```

### 4. Verify Deployment

```bash
# Check pods
kubectl get pods -n price-list

# Check service
kubectl get svc -n price-list

# Check ingress
kubectl get ingress -n price-list
```

## Troubleshooting

### Pod Issues

If pods aren't starting:
```bash
# Check pod status
kubectl describe pod <pod-name> -n price-list

# Check logs
kubectl logs <pod-name> -n price-list
```

Common issues and solutions:

1. Image Pull Errors
   ```bash
   # Check image pull status
   kubectl describe pod <pod-name> -n price-list | grep "Failed"
   
   # Add registry credentials if needed
   kubectl create secret docker-registry regcred \
     --docker-server=<your-registry-server> \
     --docker-username=<your-name> \
     --docker-password=<your-password>
   ```

2. Resource Limits
   ```bash
   # Check resource usage
   kubectl top pod <pod-name> -n price-list
   ```

3. Readiness/Liveness Probe Failures
   ```bash
   # Check probe status
   kubectl describe pod <pod-name> -n price-list | grep -A 10 "Events"
   ```

### Service Issues

```bash
# Check service details
kubectl describe svc price-list-search -n price-list

# Check endpoints
kubectl get endpoints price-list-search -n price-list
```

### Ingress Issues

```bash
# Check ingress controller
kubectl get pods -n ingress-nginx

# Check ingress logs
kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx
```

## Scaling and Monitoring

### Scale Deployment

```bash
# Scale to desired number of replicas
kubectl scale deployment price-list-search -n price-list --replicas=3

# Check scaling status
kubectl get pods -n price-list -w
```

### Monitor Resources

```bash
# Monitor pod resources
kubectl top pods -n price-list

# Monitor nodes
kubectl top nodes
```

### View Application Logs

```bash
# Stream logs from all pods
kubectl logs -f -l app=price-list-search -n price-list

# Stream logs from specific pod
kubectl logs -f <pod-name> -n price-list
```

## Maintenance

### Update Application

```bash
# Update image
kubectl set image deployment/price-list-search \
  price-list-search=your-registry/price-list-search:new-tag -n price-list

# Check update status
kubectl rollout status deployment price-list-search -n price-list
```

### Rolling Restart

```bash
# Restart all pods
kubectl rollout restart deployment price-list-search -n price-list
```

### Rollback

```bash
# Rollback to previous version
kubectl rollout undo deployment price-list-search -n price-list

# Check rollback status
kubectl rollout status deployment price-list-search -n price-list
```

## Best Practices

1. Resource Management
   - Always set resource requests and limits
   - Monitor resource usage regularly
   - Scale based on actual usage patterns

2. High Availability
   - Use multiple replicas
   - Implement proper health checks
   - Configure pod disruption budgets

3. Security
   - Use network policies
   - Implement RBAC
   - Keep container images updated

4. Monitoring
   - Set up prometheus/grafana
   - Configure alerts
   - Monitor application metrics

## Notes

- Adjust resource limits based on your application's needs
- Update probe settings based on your application's startup time
- Configure TLS in ingress for production environments
- Implement proper backup strategies for any persistent data
