---
layout: page
title: Инсталляция ArgoCD на Minikube
description: Инсталляция ArgoCD на Minikube
keywords: devops, containers, kubernetes, ci-cd, argocd, setup, minikube
permalink: /tools/containers/kubernetes/tools/ci-cd/argocd/setup/argocd-cli/
---

# Install Argo CD CLI

<br/>

Делаю:  
09.05.2023

<br/>

https://github.com/argoproj/argo-cd/releases/latest

<br/>

```
$ cd ~/tmp
$ wget https://github.com/argoproj/argo-cd/releases/download/v2.7.1/argocd-linux-amd64
$ sudo mv argocd-linux-amd64 /usr/local/bin/argocd
$ chmod +x /usr/local/bin/argocd
```
