---
layout: page
title: Инсталляция ArgoCD с помощью Helm на Minikube
description: Инсталляция ArgoCD с помощью Helm на Minikube
keywords: devops, containers, kubernetes, ci-cd, argocd, setup, minikube, helm
permalink: /tools/containers/kubernetes/tools/ci-cd/argocd/helm-installation/
---

# Инсталляция ArgoCD с помощью Helm на Minikube

**Original:**
https://gist.github.com/vfarcic/84324e2d6eb1e62e3569846a741cedea

<br/>

```
$ brew install argocd
```

<br/>

**argo/argocd-values.yaml**

<br/>

```yaml
server:
  ingress:
    enabled: true
  extraArgs:
    - --insecure
installCRDs: false
```

<br/>

```
minikube addons enable ingress

kubectl --namespace kube-system wait \
    --for=condition=ready pod \
    --selector=app.kubernetes.io/component=controller \
    --timeout=120s

export INGRESS_HOST=$(minikube ip)

###################
# Install Argo CD #
###################

git clone \
    https://github.com/vfarcic/devops-catalog-code.git

cd devops-catalog-code

git pull

helm repo add argo \
    https://argoproj.github.io/argo-helm

helm upgrade --install \
    argocd argo/argo-cd \
    --namespace argocd \
    --create-namespace \
    --version 2.8.0 \
    --set server.ingress.hosts="{argocd.$INGRESS_HOST.xip.io}" \
    --values argo/argocd-values.yaml \
    --wait

export PASS=$(kubectl --namespace argocd \
    get pods \
    --selector app.kubernetes.io/name=argocd-server \
    --output name \
    | cut -d'/' -f 2)

argocd login \
    --insecure \
    --username admin \
    --password $PASS \
    --grpc-web \
    argocd.$INGRESS_HOST.xip.io

echo $PASS

argocd account update-password
```
