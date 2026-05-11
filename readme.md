# gitops-files

Repositorio GitOps gestionado por ArgoCD. Contiene las Application CRs que apuntan al chart `pythonapps` de `belo-helm-charts` con los values específicos de cada app y ambiente.

## Estructura

```
gitops-files/
├── apps-of-apps.yaml           # Root Applications (una por ambiente)
├── gitops-core-dev/
│   ├── webserver-api01.yaml    # ArgoCD App: api01 en dev
│   └── webserver-api02.yaml    # ArgoCD App: api02 en dev
├── gitops-core-staging/
│   ├── webserver-api01.yaml
│   └── webserver-api02.yaml
├── gitops-core-production/
│   ├── webserver-api01.yaml
│   └── webserver-api02.yaml
├── gitops-core-testing/
│   ├── webserver-api01.yaml
│   └── webserver-api02.yaml
└── gitops-core-lab/
    ├── webserver-api01.yaml
    └── webserver-api02.yaml
```

## Patrón apps-of-apps

```
ArgoCD
 └── apps-of-apps.yaml
      ├── gitops-core-dev       → gitops-core-dev/
      │    ├── webserver-api01-dev  → pythonapps chart + values dev
      │    └── webserver-api02-dev  → pythonapps chart + values dev
      ├── gitops-core-staging   → gitops-core-staging/
      │    └── ...
      └── gitops-core-production → gitops-core-production/
           └── ...
```

## Bootstrap

```bash
# 1. Registrar el repo de charts en ArgoCD (si es privado)
argocd repo add https://github.com/Valentino-33/belo-helm-charts --username <user> --password <token>

# 2. Registrar este repo
argocd repo add https://github.com/Valentino-33/gitops-files --username <user> --password <token>

# 3. Aplicar root application
kubectl apply -f apps-of-apps.yaml -n argocd

# 4. ArgoCD descubre y sincroniza el resto automáticamente
argocd app sync gitops-core-dev
```

## Ambientes

| Ambiente | Namespace | Auto-sync | Observaciones |
|----------|-----------|-----------|---------------|
| dev | dev | si | Desplegado automáticamente en cada tag push |
| staging | staging | si | Promovido manualmente desde dev |
| production | production | no | Requiere aprobación manual + prune=false |
| testing | testing | si | Para pruebas de integración |
| lab | lab | si | Para experimentación libre |

## Flujo de actualización de imagen

Tekton (`task-bump-gitops`) actualiza `image.tag` en `belo-helm-charts/pythonapps/apps/<app>/<env>/values-*.yaml` y hace commit. ArgoCD detecta el cambio en `belo-helm-charts` y reconcilia el Rollout en el clúster.

No hay cambios directos en este repo durante el ciclo CI/CD normal — solo las Application CRs que definen qué desplegar y dónde.
