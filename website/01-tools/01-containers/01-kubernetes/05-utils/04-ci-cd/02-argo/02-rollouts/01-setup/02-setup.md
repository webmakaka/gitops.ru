---
layout: page
title: Инсталляция Argo Rollouts в kind
description: Инсталляция Argo Rollouts в kind
keywords: devops, containers, kubernetes, argo, rollouts, setup, minikube
permalink: /tools/containers/kubernetes/utils/ci-cd/argo/rollouts/setup/
---

# Инсталляция Argo Rollouts в kind

<br/>

Делаю:  
2024.12.21

<br/>

```
$ kubectl create namespace argo-rollouts
$ kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml
```

<br/>

```
$ kubectl api-resources | grep -i argo
```

<br/>

```
$ kubectl get pods -n argo-rollouts
NAME                             READY   STATUS    RESTARTS   AGE
argo-rollouts-6f4f78ffd8-n6425   1/1     Running   0          65s
```
