# Price List Search - Kubernetes Deployment Guide

This guide explains how to deploy the Price List Search application to a Kubernetes cluster.

## Prerequisites

- Kubernetes cluster
- kubectl configured to access your cluster
- Docker registry access
- Domain name (for ingress configuration)

## Directory Structure

```
/
├── src/                    # Application source code
│   ├── app/               # Next.js app directory
│   │   ├── page.tsx      # Main page component
│   │   ├── layout.tsx    # Root layout
│   │   └── globals.css   # Global styles
│   ├── components/       # React components
│   │   ├── ExcelSearchComponent.tsx  # Main search component
│   │   └── ui/          # UI components
│   └── lib/             # Utility functions
│       └── excel.ts     # Excel parsing logic
├── k8s/                 # Kubernetes configurations
│   ├── deployment.yaml  # Deployment configuration
│   ├── service.yaml     # Service configuration
│   ├── ingress.yaml    # Ingress configuration
│   └── README.md       # This file
├── public/             # Static files
├── Dockerfile          # Container configuration
├── package.json        # Project dependencies
└── next.config.ts     # Next.js configuration
```

## Application Setup

### Requirements

- Node.js 18.x or later
- npm 9.x or later
- Git

### Local Development Setup

1. Clone the repository:
   ```bash
   git clone <repository-url>
   cd price-list-search
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Create a `.env.local` file:
   ```bash
   # .env.local
   PORT=8000
   ```

4. Development mode:
   ```bash
   npm run dev
   # Application will be available at http://localhost:8000
   ```

5. Build for production:
   ```bash
   npm run build
   ```

6. Start production server:
   ```bash
   npm start
   ```

### Environment Variables

- `PORT`: Application port (default: 8000)

### Application Components

#### Main Components

1. **Excel Search Component** (`src/components/ExcelSearchComponent.tsx`)
   - Handles file upload and parsing
   - Provides real-time search functionality
   - Manages selected items and price calculations

2. **Page Layout** (`src/app/page.tsx`)
   - Main application layout
   - Header and footer components
   - Container for the Excel search component

3. **Excel Parser** (`src/lib/excel.ts`)
   - Handles Excel file parsing
   - Converts Excel data to JSON format
   - Validates file format and content

#### Key Features

- Real-time search through Excel data
- Drag and drop file upload
- Price list item selection
- Total price calculation
- Responsive design
- Accessibility support

### Testing

Run the test suite:
```bash
npm test
```

Run tests in watch mode:
```bash
npm run test:watch
```

### Development Guidelines

1. Code Style
   - Follow TypeScript best practices
   - Use ESLint for code linting
   - Follow Next.js 13+ conventions

2. Component Structure
   - Keep components focused and single-responsibility
   - Use TypeScript interfaces for props
   - Implement proper error handling

3. Performance
   - Implement proper memoization where needed
   - Use proper loading states
   - Handle large datasets efficiently

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
