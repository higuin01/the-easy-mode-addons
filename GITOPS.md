# Estrutura GitOps - App of Apps

## Estrutura Final

```
the-easy-mode-addons/
├── installed-addons/
│   ├── argocd-apps/              # ← ArgoCD Applications (GitOps)
│   │   ├── root-app.yaml         # ← App of Apps (bootstrap)
│   │   ├── ingress-nginx.yaml    # ← Application para ingress
│   │   ├── prometheus.yaml       # ← Application para prometheus
│   │   ├── loki.yaml
│   │   ├── tempo.yaml
│   │   ├── mimir.yaml
│   │   ├── minio.yaml
│   │   ├── otel-operator.yaml
│   │   └── metrics-server.yaml
│   │
│   ├── ingress-nginx/            # ← Helm chart
│   │   ├── Chart.yaml
│   │   └── values.yaml
│   ├── prometheus/
│   │   ├── Chart.yaml
│   │   └── values.yaml
│   └── ...
│
└── dev-addons/                   # ← Desenvolvimento
```

## Fluxo de Bootstrap

```
┌─────────────────────────────────────────────────────────────┐
│ 1. make install-argocd-apps                                 │
│    (aplica root-app.yaml uma única vez)                     │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. root-app monitora: installed-addons/argocd-apps/        │
│    (detecta todos os .yaml nessa pasta)                     │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. Cria Applications: ingress-nginx, prometheus, loki...   │
│    (cada .yaml vira uma Application no ArgoCD)              │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│ 4. Cada Application instala seu Helm chart                 │
│    (lê installed-addons/ADDON/Chart.yaml + values.yaml)    │
└─────────────────────────────────────────────────────────────┘
```

## GitOps em Ação

### Adicionar Novo Addon

```bash
# 1. Criar estrutura
mkdir installed-addons/grafana
cat > installed-addons/grafana/Chart.yaml
cat > installed-addons/grafana/values.yaml

# 2. Criar Application
cat > installed-addons/argocd-apps/grafana.yaml <<EOF
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: grafana
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/USER/the-easy-mode-addons
    targetRevision: main
    path: installed-addons/grafana
  destination:
    server: https://kubernetes.default.svc
    namespace: monitoring
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
EOF

# 3. Git push
git add .
git commit -m "Add Grafana"
git push

# 4. ArgoCD detecta e instala automaticamente! 🎉
```

### Atualizar Addon Existente

```bash
# 1. Editar values
vim installed-addons/prometheus/values.yaml

# 2. Git push
git commit -am "Update Prometheus retention to 60d"
git push

# 3. ArgoCD aplica automaticamente! 🎉
```

### Remover Addon

```bash
# 1. Deletar Application
rm installed-addons/argocd-apps/minio.yaml

# 2. Git push
git commit -am "Remove MinIO"
git push

# 3. ArgoCD remove do cluster automaticamente! 🎉
```

## Vantagens

✅ **Uma única aplicação manual**: Só `root-app.yaml`
✅ **Tudo mais é Git**: Commit = Deploy
✅ **Auditável**: Histórico completo no Git
✅ **Rollback fácil**: `git revert`
✅ **Disaster Recovery**: Reinstalar cluster = tudo volta
