# The Easy Mode - Kubernetes Addons

Repositório de addons não críticos para o cluster Kubernetes Easy Mode, gerenciados via ArgoCD.

## Estrutura

```
the-easy-mode-addons/
├── installed-addons/     # Addons gerenciados pelo ArgoCD (produção)
│   ├── argocd-apps/     # Applications do ArgoCD (GitOps)
│   │   ├── root-app.yaml          # App of Apps (bootstrap)
│   │   ├── ingress-nginx.yaml
│   │   ├── prometheus.yaml
│   │   └── ...
│   ├── ingress-nginx/   # Helm chart
│   │   ├── Chart.yaml
│   │   └── values.yaml
│   ├── prometheus/
│   └── ...
└── dev-addons/          # Addons em desenvolvimento/teste
```

## Addons Instalados

### Observabilidade
- **Prometheus Stack**: Métricas e alertas
- **Loki**: Agregação de logs
- **Tempo**: Distributed tracing
- **Mimir**: Long-term metrics storage
- **OpenTelemetry Operator**: Instrumentação automática

### Infraestrutura
- **NGINX Ingress**: Ingress controller
- **MinIO**: Object storage (S3-compatible)
- **Metrics Server**: Métricas de recursos do Kubernetes

## Como Usar

### Instalação via ArgoCD (App of Apps)

Após o cluster estar criado com os addons críticos:

```bash
# Aplicar root-app (uma única vez)
make install-argocd-apps

# Ou manualmente:
kubectl apply -f https://raw.githubusercontent.com/YOUR_USERNAME/the-easy-mode-addons/main/installed-addons/argocd-apps/root-app.yaml
```

A `root-app` criará automaticamente todas as outras Applications.

### GitOps Workflow

1. Edite values.yaml ou adicione novo addon
2. `git commit` e `git push`
3. ArgoCD detecta e aplica automaticamente! 🎉

### Desenvolvimento de Novos Addons

1. Crie uma pasta em `dev-addons/` com o nome do addon
2. Adicione `Chart.yaml`, `values.yaml` e `application.yaml`
3. Teste localmente
4. Mova para `installed-addons/` quando estável

## Configuração

Cada addon possui:
- `Chart.yaml`: Definição do chart Helm
- `values.yaml`: Valores customizados
- `application.yaml`: Manifesto ArgoCD Application

## Requisitos

- Cluster Kubernetes com ArgoCD instalado
- Acesso ao repositório Git configurado no ArgoCD
- Addons críticos já instalados (Cilium, MetalLB, ArgoCD)
