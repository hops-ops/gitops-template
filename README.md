# GitOps Repository

This repository is managed by [ArgoCD](https://argo-cd.readthedocs.io/) and deployed by the [hops GitopsStack](https://github.com/hops-ops/gitops-stack).

## How It Works

ArgoCD watches this repo and automatically syncs changes to your cluster. The structure follows an **app-of-apps** pattern:

```
apps/                                  <- Drop Application manifests here
crossplane/                            <- Crossplane Configuration packages and resources
```

## Adding an Application

Create a YAML file in `apps/` with an ArgoCD Application manifest:

```yaml
# apps/my-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/my-org/my-app.git
    path: k8s/              # path to manifests in the source repo
    targetRevision: HEAD
  destination:
    server: https://kubernetes.default.svc
    namespace: my-app
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
    syncOptions:
    - CreateNamespace=true
```

Commit and push — ArgoCD picks it up automatically.

### Helm Chart Example

```yaml
# apps/ingress-nginx.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: ingress-nginx
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://kubernetes.github.io/ingress-nginx
    chart: ingress-nginx
    targetRevision: 4.12.1
    helm:
      valuesObject:
        controller:
          replicaCount: 2
  destination:
    server: https://kubernetes.default.svc
    namespace: ingress-nginx
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
    syncOptions:
    - CreateNamespace=true
```

## What ArgoCD Syncs

| ArgoCD Application | Watches | Purpose |
|--------------------|---------|---------|
| `apps` | `apps/` | Your applications (app-of-apps root) |
| `crossplane` | `crossplane/` | Crossplane Configuration packages and resources (optional) |

## Environments

For multiple environments, use separate directories per environment and create an Application per environment:

```
apps/
  staging/
    my-app.yaml           # points to staging branch/values
  production/
    my-app.yaml           # points to production branch/values
```

Or use [ApplicationSets](https://argo-cd.readthedocs.io/en/latest/user-guide/application-set/) for automatic environment generation from folder structure.
