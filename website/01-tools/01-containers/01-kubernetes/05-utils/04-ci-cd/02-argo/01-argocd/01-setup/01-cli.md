---
layout: page
title: Инсталляция ArgoCD CLI
description: Инсталляция ArgoCD CLI
keywords: tools, containers, kubernetes, ci-cd, argocd, setup, cli, minikube
permalink: /tools/containers/kubernetes/utils/ci-cd/argo/argocd/setup/cli/
---

# Инсталляция ArgoCD CLI

<br/>

Делаю:  
2024.03.25

<br/>

https://github.com/argoproj/argo-cd/releases/latest

<br/>

```
$ cd ~/tmp
$ wget https://github.com/argoproj/argo-cd/releases/download/v2.10.2/argocd-linux-amd64
$ sudo mv argocd-linux-amd64 /usr/local/bin/argocd
$ chmod +x /usr/local/bin/argocd
```

<br/>

```
$ argocd version
argocd: v2.10.2+fcf5d8c
  BuildDate: 2024-03-01T21:47:51Z
  GitCommit: fcf5d8c2381b68ab1621b90be63913b12cca2eb7
  GitTreeState: clean
  GoVersion: go1.21.7
  Compiler: gc
  Platform: linux/amd64
FATA[0000] Argo CD server address unspecified
```
