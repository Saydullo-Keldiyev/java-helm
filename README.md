# Java Application Helm Chart

A Helm chart for deploying Java applications on Kubernetes.

## Overview

This Helm chart provides a production-ready deployment configuration for Java applications. It includes:

- Kubernetes Deployment with configurable replicas
- Service for internal cluster communication
- Optional Ingress for external access
- Configurable JVM options
- Health checks (liveness and readiness probes)
- Resource limits and requests

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+

## Installation

### Add the chart repository (if applicable)

```bash
helm repo add my-repo https://your-chart-repository.com
helm repo update
```

### Install from local directory

```bash
helm install my-java-app ./java-app-chart
```

### Install with custom values

```bash
helm install my-java-app ./java-app-chart \
  --set image.repository=your-registry/your-java-app \
  --set image.tag=1.0.0 \
  --set replicaCount=3
```

### Install using a custom values file

```bash
helm install my-java-app ./java-app-chart -f custom-values.yaml
```

## Uninstallation

```bash
helm uninstall my-java-app
```

## Configuration

The following table lists the configurable parameters and their default values.

### General

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of pod replicas | `1` |
| `nameOverride` | Override the chart name | `""` |
| `fullnameOverride` | Override the full resource name | `""` |

### Image

| Parameter | Description | Default |
|-----------|-------------|---------|
| `image.repository` | Container image repository | `openjdk` |
| `image.tag` | Container image tag | `17-jdk-slim` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |

### Service

| Parameter | Description | Default |
|-----------|-------------|---------|
| `service.type` | Kubernetes service type | `ClusterIP` |
| `service.port` | Service port | `8080` |

### Ingress

| Parameter | Description | Default |
|-----------|-------------|---------|
| `ingress.enabled` | Enable Ingress | `false` |
| `ingress.className` | Ingress class name | `""` |
| `ingress.annotations` | Ingress annotations | `{}` |
| `ingress.hosts` | List of Ingress hosts | See values.yaml |
| `ingress.tls` | TLS configuration | `[]` |

### Java Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `java.opts` | JVM options passed via JAVA_OPTS | `-Xms256m -Xmx512m` |

### Resources

| Parameter | Description | Default |
|-----------|-------------|---------|
| `resources.limits.cpu` | CPU limit | `500m` |
| `resources.limits.memory` | Memory limit | `512Mi` |
| `resources.requests.cpu` | CPU request | `250m` |
| `resources.requests.memory` | Memory request | `256Mi` |

### Health Probes

| Parameter | Description | Default |
|-----------|-------------|---------|
| `livenessProbe.httpGet.path` | Liveness probe endpoint | `/actuator/health` |
| `livenessProbe.initialDelaySeconds` | Initial delay before liveness check | `60` |
| `livenessProbe.periodSeconds` | Liveness check interval | `10` |
| `readinessProbe.httpGet.path` | Readiness probe endpoint | `/actuator/health/readiness` |
| `readinessProbe.initialDelaySeconds` | Initial delay before readiness check | `30` |
| `readinessProbe.periodSeconds` | Readiness check interval | `5` |

### Environment Variables

| Parameter | Description | Default |
|-----------|-------------|---------|
| `env` | List of environment variables | `[]` |

### Scheduling

| Parameter | Description | Default |
|-----------|-------------|---------|
| `nodeSelector` | Node selector labels | `{}` |
| `tolerations` | Pod tolerations | `[]` |
| `affinity` | Pod affinity rules | `{}` |

## Examples

### Basic deployment with custom image

```yaml
image:
  repository: my-registry/my-java-app
  tag: "2.0.0"

replicaCount: 2

resources:
  limits:
    cpu: 1000m
    memory: 1Gi
  requests:
    cpu: 500m
    memory: 512Mi
```

### Production deployment with Ingress

```yaml
image:
  repository: my-registry/my-java-app
  tag: "2.0.0"

replicaCount: 3

ingress:
  enabled: true
  className: nginx
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
  hosts:
    - host: api.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: api-tls
      hosts:
        - api.example.com

env:
  - name: SPRING_PROFILES_ACTIVE
    value: "production"
  - name: DATABASE_URL
    valueFrom:
      secretKeyRef:
        name: db-credentials
        key: url

java:
  opts: "-Xms512m -Xmx1024m -XX:+UseG1GC"
```

### Custom health check endpoints

```yaml
livenessProbe:
  httpGet:
    path: /health/live
    port: http
  initialDelaySeconds: 45
  periodSeconds: 15

readinessProbe:
  httpGet:
    path: /health/ready
    port: http
  initialDelaySeconds: 20
  periodSeconds: 5
```

## Chart Structure

```
java-app-chart/
├── Chart.yaml           # Chart metadata
├── values.yaml          # Default configuration values
├── README.md            # This file
└── templates/
    ├── _helpers.tpl     # Template helper functions
    ├── deployment.yaml  # Kubernetes Deployment
    ├── service.yaml     # Kubernetes Service
    ├── ingress.yaml     # Kubernetes Ingress (optional)
    └── NOTES.txt        # Post-installation notes
```

## Upgrading

To upgrade an existing release:

```bash
helm upgrade my-java-app ./java-app-chart -f custom-values.yaml
```

To perform a dry-run before upgrading:

```bash
helm upgrade my-java-app ./java-app-chart -f custom-values.yaml --dry-run
```

## Troubleshooting

### View deployed resources

```bash
kubectl get all -l app.kubernetes.io/instance=my-java-app
```

### Check pod logs

```bash
kubectl logs -l app.kubernetes.io/instance=my-java-app -f
```

### Describe pod for debugging

```bash
kubectl describe pod -l app.kubernetes.io/instance=my-java-app
```

### Validate chart templates

```bash
helm template my-java-app ./java-app-chart --debug
```

## License

This Helm chart is provided under the MIT License.
