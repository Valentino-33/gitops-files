# Guía rápida de comandos — gitops-files

> Este repo no tiene Makefile. Los comandos son `kubectl` y `argocd` CLI directo.
> Prerrequisito: `kubectl` apuntando al cluster y ArgoCD instalado.

---

## Bootstrap (primera vez o re-deploy)

```bash
# 1. (Opcional) Registrar repos privados en ArgoCD
argocd repo add https://github.com/Valentino-33/belo-helm-charts \
  --username <user> --password <token>
argocd repo add https://github.com/Valentino-33/gitops-files \
  --username <user> --password <token>

# 2. Aplicar la root Application (apps-of-apps)
kubectl apply -f apps-of-apps.yaml -n argocd

# 3. Verificar que los cores se descubrieron
kubectl -n argocd get applications

# 4. Sincronizar manualmente si auto-sync no está activo
argocd app sync gitops-core-dev
argocd app sync gitops-core-staging
```

---

## Operaciones frecuentes

```bash
# Ver estado de todas las applications
kubectl -n argocd get applications
argocd app list

# Ver detalles de una app específica
argocd app get webserver-api01-dev

# Sincronizar una app manualmente
argocd app sync webserver-api01-dev

# Forzar re-sync (útil cuando ArgoCD quedó desfasado)
argocd app sync webserver-api01-dev --force

# Rollback a la revision anterior
argocd app rollback webserver-api01-dev

# Ver historial de sincronizaciones
argocd app history webserver-api01-dev
```

---

## Estructura del repo

```
gitops-files/
├── apps-of-apps.yaml               ← bootstrap: aplicar una sola vez
├── gitops-core-dev/
│   ├── webserver-api01.yaml        ← Application ArgoCD: api01 en dev
│   └── webserver-api02.yaml
├── gitops-core-staging/
├── gitops-core-production/
├── gitops-core-testing/
└── gitops-core-lab/
```

---

## Flujo de actualización de imagen

Tekton actualiza `image.tag` en `belo-helm-charts` → ArgoCD detecta el commit → sincroniza el Rollout.
**No tocar este repo durante el ciclo normal de CI/CD** — es solo para definir qué desplegar y dónde.

Para actualizar manualmente la imagen de api01 en dev:
```bash
# Editar en belo-helm-charts (NO en gitops-files):
# pythonapps/apps/webserver-api01/dev/values-webserver-api01-dev.yaml
# Cambiar image.tag y hacer commit → ArgoCD detecta y sincroniza
```

---

## Ambientes y sus políticas de sync

| Core | Auto-sync | Prune | SelfHeal | Notas |
|------|-----------|-------|----------|-------|
| gitops-core-dev | sí | sí | sí | Cualquier commit en belo-helm-charts despliega automáticamente |
| gitops-core-staging | sí | sí | sí | Promovido desde dev |
| gitops-core-production | no | no | no | Requiere aprobación manual |
| gitops-core-testing | sí | sí | sí | Para smoke tests de CI |
| gitops-core-lab | sí | sí | sí | Para experimentación |
