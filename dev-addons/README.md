# Dev Addons

Pasta para desenvolvimento e teste de novos addons antes de movê-los para `installed-addons/`.

## Estrutura de um Addon

Cada addon deve ter:

```
addon-name/
├── Chart.yaml          # Definição do Helm chart
├── values.yaml         # Valores customizados
└── application.yaml    # Manifesto ArgoCD Application
```

## Exemplo: Chart.yaml

```yaml
apiVersion: v2
name: my-addon
description: My Custom Addon
type: application
version: 1.0.0
dependencies:
  - name: chart-name
    version: 1.2.3
    repository: https://charts.example.com
```

## Exemplo: values.yaml

```yaml
# Custom values for the chart
replicaCount: 1
service:
  type: ClusterIP
  port: 80
```

## Exemplo: application.yaml

```yaml
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
    namespace: my-namespace
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

## Workflow

1. Crie pasta com nome do addon
2. Adicione os 3 arquivos acima
3. Teste localmente: `kubectl apply -f dev-addons/my-addon/application.yaml`
4. Quando estável, mova para `installed-addons/`
5. Commit e push - ArgoCD sincroniza automaticamente
