# GitOps Monorepo - Padrão Codefresh

## 🎯 Estrutura apps/*/envs/*

Este repositório segue o padrão Codefresh recomendado para descoberta automática de aplicações.

```
apps/
├── webapp1/                    # Frontend React
│   └── envs/
│       ├── dev/               # Kustomize
│       ├── hom/               # Kustomize
│       └── prod/              # Kustomize
└── api-service/               # Backend API
    └── envs/
        ├── dev/               # Helm
        ├── hom/               # Helm
        └── prod/              # Helm
```

## 🔍 Descoberta Automática

O `monorepo-applicationset.yaml` usa Git generator para descobrir automaticamente:
- Qualquer pasta seguindo `apps/*/envs/*`
- Suporte automático para Helm, Kustomize, plain YAML
- Namespace automático: `{app}-{env}`
- Nome da Application: `{app}-{env}-{controlPlane}`

## 📝 Como Adicionar Nova App

### 1. Criar estrutura
```bash
mkdir -p apps/nova-app/envs/{dev,hom,prod}
```

### 2. Adicionar manifestos
- **Kustomize**: `kustomization.yaml` + recursos
- **Helm**: `Chart.yaml` + `values.yaml`
- **Plain YAML**: arquivos `.yaml` diretamente

### 3. Commit e Push
O ApplicationSet descobrirá automaticamente e criará as Applications.

## 🎛️ Exemplos Implementados

### webapp1 (Kustomize)
- **DEV**: 1 réplica, recursos mínimos
- **HOM**: 2 réplicas, recursos intermediários  
- **PROD**: 3 réplicas, HA, versão específica

### api-service (Helm)
- **DEV**: PostgreSQL embedded, sem persistência
- **HOM**: PostgreSQL externo, backup diário
- **PROD**: RDS, alta disponibilidade

## ✅ Melhores Práticas

### Naming Convention
- Apps: kebab-case (`api-service`, `webapp1`)
- Environments: `dev`, `hom`, `prod`
- Namespaces: `{app}-{env}` (`webapp1-dev`)

### Resource Management
- **DEV**: Recursos mínimos, sem persistência
- **HOM**: Recursos intermediários, backup
- **PROD**: HA, monitoramento, alertas

### Security
- Secrets via External Secrets Operator
- Network Policies por namespace
- Pod Security Standards

## 🚀 Progressive Rollout

O ApplicationSet implementa rollout progressivo:
- **DEV**: 100% automático
- **HOM**: 25% gradual
- **PROD**: 0% manual

## 🔧 Troubleshooting

### App não descoberta
```bash
# Verificar estrutura
ls -la apps/*/envs/*

# Verificar ApplicationSet
kubectl describe applicationset monorepo-apps -n argocd
```

### Sync failures
```bash
# Ver logs da Application
argocd app logs {app}-{env}-{cp}

# Forçar sync
argocd app sync {app}-{env}-{cp}
```