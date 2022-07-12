---
layout: page
title: ArgoCD
description: ArgoCD
keywords: linux, kubernetes, FluxCD
permalink: /study/videos/tools/containers/kubernetes/ci-cd/argocd/
---

# ArgoCD

<br/>

### ArgoCD Tutorial for Beginners | GitOps CD for Kubernetes

https://www.youtube.com/watch?v=MeU5_k9ssrs

**GitLab:**  
https://gitlab.com/nanuchi/argocd-app-config/

<br/>

**Делаю:**  
11.11.2021

<br/>

```
// install ArgoCD in k8s
$ kubectl create namespace argocd
$ kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

$ kubectl -n argocd get pods

// Получить пароль администратора:
$ kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode && echo

// access ArgoCD UI
$ kubectl get svc -n argocd
$ kubectl port-forward svc/argocd-server 8080:443 -n argocd
```

<br/>

localhost:8080

<br/>

### Configure Argo CD

```
$ cd ~/tmp
$ git clone https://gitlab.com/nanuchi/argocd-app-config/
$ cd argocd-app-config/
```

<br/>

Добавляем application.yaml

<br/>

$ git add /git commit / git push
$ kubectl apply -f application.yaml

<br/>

Смотрим UI

<br/>

### Test sync

Deployment
Меняем image 1.2
Смотрим результат
