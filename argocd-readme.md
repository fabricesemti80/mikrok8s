#

## Deployment

```bash

# crete namespace
kubectl create namespace argocd

# add repo
helm repo add argo https://argoproj.github.io/argo-helm
## "argo" has been added to your repositories

# deploy helm release
helm install argocd argo/argo-cd -n argocd --values=helm/argo-values.yml

# get the password of the user
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

```