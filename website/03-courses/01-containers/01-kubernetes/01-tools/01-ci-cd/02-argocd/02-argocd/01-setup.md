---
layout: page
title: Инсталляция ArgoCD на Minikube
description: Инсталляция ArgoCD на Minikube
keywords: devops, containers, kubernetes, ci-cd, argocd, setup, minikube
permalink: /devops/containers/kubernetes/ci-cd/argocd/setup/
---

# Инсталляция ArgoCD на Minikube

**Original:**
https://gist.github.com/vfarcic/84324e2d6eb1e62e3569846a741cedea

<br/>

https://argo-cd.readthedocs.io/en/stable/getting_started/

<br/>

```
$ brew install argocd
```

<br/>

```
$ kubectl create namespace argocd
$ kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

<br/>

```
$ kubectl -n argocd get pods
NAME                                 READY   STATUS    RESTARTS   AGE
argocd-application-controller-0      1/1     Running   0          111s
argocd-dex-server-5fbb579948-lbfg8   1/1     Running   0          111s
argocd-redis-6fb68d9df5-tqm4w        1/1     Running   0          111s
argocd-repo-server-b4c6dc8f9-pqpns   1/1     Running   0          111s
argocd-server-56ffccb4cd-p2ds4       1/1     Running   0          111s
```

<br/>

```
$ kubectl port-forward svc/argocd-server -n argocd 8080:443
```

<br/>

### Получаем пароль

```
$ kubectl get pods -n argocd -l app.kubernetes.io/name=argocd-server -o name | cut -d'/' -f 2
```

<br/>

localhost:8080

admin / результат выполнения команды выше.
