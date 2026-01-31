# Kubernetes Easy Mode - Addons

Non-critical addons managed by ArgoCD for the Kubernetes Easy Mode cluster.

## Structure

```
the-easy-mode-addons/
├── installed-addons/       # Production addons (ArgoCD synced)
│   ├── argocd-apps/        # ArgoCD Application manifests
│   ├── ingress-nginx/      # NGINX Ingress Controller
│   ├── prometheus/         # Prometheus + Grafana
│   └── metrics-server/     # Metrics Server
│
└── dev-addons/             # Development/testing addons
    ├── argocd-apps/
    ├── loki/               # Log aggregation
    ├── tempo/              # Distributed tracing
    ├── mimir/              # Long-term metrics
    ├── minio/              # S3-compatible storage
    └── otel-operator/      # OpenTelemetry
```

## Installed Addons (Production)

### NGINX Ingress Controller
- **Purpose**: HTTP/HTTPS routing
- **Type**: LoadBalancer (MetalLB)
- **Chart**: ingress-nginx/ingress-nginx

### Prometheus Stack
- **Purpose**: Monitoring + Grafana dashboards
- **Components**: Prometheus, Grafana, Alertmanager
- **Storage**: 10Gi (Prometheus), 5Gi (Grafana)
- **Chart**: prometheus-community/kube-prometheus-stack
- **Note**: CRDs installed separately via prometheus-crds app

### Metrics Server
- **Purpose**: Resource metrics for HPA
- **Chart**: metrics-server/metrics-server

## Dev Addons (Optional)

### Loki
- **Purpose**: Log aggregation
- **Storage**: MinIO backend
- **Chart**: grafana/loki

### Tempo
- **Purpose**: Distributed tracing
- **Storage**: MinIO backend
- **Chart**: grafana/tempo

### Mimir
- **Purpose**: Long-term metrics storage
- **Storage**: MinIO backend
- **Chart**: grafana/mimir-distributed

### MinIO
- **Purpose**: S3-compatible object storage
- **Use**: Backend for Loki, Tempo, Mimir
- **Chart**: minio/minio

### OpenTelemetry Operator
- **Purpose**: Auto-instrumentation for traces
- **Components**: Operator, Collector
- **Chart**: open-telemetry/opentelemetry-operator

## Installation

### Prerequisites
- Kubernetes cluster with ArgoCD installed
- `kubectl` configured

### Install Production Addons

```bash
# From the-easy-mode-k8s repo
make install-argocd-apps
```

Or manually:
```bash
kubectl apply -f installed-addons/argocd-apps/
```

### Install Dev Addons

```bash
kubectl apply -f dev-addons/argocd-apps/
```

## Adding New Addons

### 1. Create Addon Structure

```bash
cd dev-addons
mkdir my-addon
cd my-addon
```

### 2. Create Chart.yaml

```yaml
apiVersion: v2
name: my-addon
description: My Custom Addon
type: application
version: 1.0.0
dependencies:
  - name: upstream-chart
    version: 1.2.3
    repository: https://charts.example.com
```

### 3. Create values.yaml

```yaml
upstream-chart:
  replicas: 3
  resources:
    limits:
      memory: 512Mi
```

### 4. Create ArgoCD Application

```bash
cd ../argocd-apps
```

```yaml
# my-addon.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-addon
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/YOUR_USERNAME/the-easy-mode-addons
    targetRevision: main
    path: dev-addons/my-addon
  destination:
    server: https://kubernetes.default.svc
    namespace: my-addon
  syncPolicy:
    syncOptions:
      - CreateNamespace=true
      - ServerSideApply=true
    automated:
      prune: true
      selfHeal: true
```

### 5. Test

```bash
kubectl apply -f argocd-apps/my-addon.yaml
```

### 6. Promote to Production

```bash
# Move to installed-addons
mv dev-addons/my-addon installed-addons/
mv dev-addons/argocd-apps/my-addon.yaml installed-addons/argocd-apps/

# Update path in application.yaml
sed -i 's|dev-addons|installed-addons|g' installed-addons/argocd-apps/my-addon.yaml

# Commit
git add installed-addons/
git commit -m "Promote my-addon to production"
git push
```

## Custom Manifests

To add custom Kubernetes manifests to a Helm-based addon:

```bash
cd installed-addons/my-addon
mkdir templates
```

Add your manifests:
```yaml
# templates/custom-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-custom-ingress
spec:
  # ...
```

ArgoCD will include these when rendering the Helm chart.

## Troubleshooting

### App stuck in "Progressing"

```bash
# Check app details
kubectl describe application <app-name> -n argocd

# Force refresh
kubectl patch application <app-name> -n argocd \
  --type merge -p '{"operation":{"sync":{"revision":"HEAD"}}}'
```

### CRDs not installing

Some charts (like Prometheus) have large CRDs that need separate installation:

```yaml
# Create separate CRDs app with sync-wave: "-1"
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "-1"
```

### Sync loop (OutOfSync constantly)

Add `ignoreDifferences` for fields modified by operators:

```yaml
spec:
  ignoreDifferences:
  - group: admissionregistration.k8s.io
    kind: ValidatingWebhookConfiguration
    jqPathExpressions:
    - .webhooks[]?.clientConfig.caBundle
```

## GitOps Best Practices

1. **Always commit changes to Git first**
   - Never `kubectl apply` directly
   - Let ArgoCD sync from Git

2. **Use sync-waves for dependencies**
   ```yaml
   annotations:
     argocd.argoproj.io/sync-wave: "-1"  # Install first
   ```

3. **Enable ServerSideApply for large resources**
   ```yaml
   syncOptions:
     - ServerSideApply=true
   ```

4. **Test in dev-addons before promoting**
   - Validate in isolated environment
   - Move to installed-addons when stable

5. **Use ignoreDifferences sparingly**
   - Only for fields modified by operators
   - Document why each field is ignored

## Related Documentation

- [Main Cluster Repo](https://github.com/YOUR_USERNAME/the-easy-mode-k8s)
- [GitOps Guide](GITOPS.md)
- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)

## License

MIT
