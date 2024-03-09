---
layout: page
title: Инсталляция ArgoCD на Minikube
description: Инсталляция ArgoCD на Minikube
keywords: devops, containers, kubernetes, ci-cd, argocd, setup, minikube
permalink: /tools/containers/kubernetes/tools/ci-cd/argocd/setup/
---

# Инсталляция ArgoCD на Minikube

<br/>

Делаю:  
2024.03.09

<br/>

**Original:**
https://gist.github.com/vfarcic/84324e2d6eb1e62e3569846a741cedea

<br/>

https://argo-cd.readthedocs.io/en/stable/getting_started/

<br/>

### [Install Argo CD CLI](/tools/containers/kubernetes/tools/ci-cd/argocd/setup/argocd-cli/)

<br/>

```
$ kubectl create namespace argocd
$ kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

<br/>

```
$ kubectl -n argocd get pods
NAME                                                READY   STATUS    RESTARTS   AGE
argocd-application-controller-0                     1/1     Running   0          74s
argocd-applicationset-controller-5478c64d7c-2pbqd   1/1     Running   0          74s
argocd-dex-server-6b576d67c9-z5qqh                  1/1     Running   0          74s
argocd-notifications-controller-5f6c747849-9l5sw    1/1     Running   0          74s
argocd-redis-76748db5f4-w4rjg                       1/1     Running   0          74s
argocd-repo-server-58c78bd74f-228dm                 1/1     Running   0          74s
argocd-server-5fd847d6bc-28frv                      1/1     Running   0          74s
```

<br/>

```
$ kubectl port-forward svc/argocd-server -n argocd 8080:443
```

<br/>

### Получаем пароль

<br/>

```
// Получить пароль для входа
$ kubectl -n argocd get secret argocd-initial-admin-secret \
          -o jsonpath="{.data.password}" | base64 -d; echo
```

<br/>

```
// admin / результат выполнения команды выше.
localhost:8080
```
