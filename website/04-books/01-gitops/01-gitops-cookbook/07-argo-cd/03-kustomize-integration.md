---
layout: page
title: GitOps Cookbook - Argo CD - Kustomize Integration
description: GitOps Cookbook - Argo CD - Kustomize Integration
keywords: books, gitops, argo-cd, Kustomize Integration
permalink: /books/gitops/gitops-cookbook/argo-cd/kustomize-integration/
---

<br/>

# [Book] [OK!] 7.3 Kustomize Integration

<br/>

Делаю:  
2024.03.24

<br/>

```yaml
$ cat << 'EOF' | kubectl create -f -
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: bgdk-app
  namespace: argocd
spec:
  destination:
    namespace: bgdk
    server: https://kubernetes.default.svc
  project: default
  source:
    path: ch07/bgdk/bgdk
    repoURL: https://github.com/wildmakaka/gitops-cookbook-sc.git
    targetRevision: main
  syncPolicy:
    automated: {}
EOF
```

<br/>

```
$ kubectl patch svc bgd -n bgdk -p '{"spec": {"type": "NodePort"}}'
```

<br/>

```
$ kubectl get services -n bgdk
NAME   TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
bgd    NodePort   10.105.199.181   <none>        8080:31257/TCP   3m11s
```

<br/>

```
// [OK!]
http://192.168.49.2:31257
```

<br/>

gitops-cookbook-sc/ch07/bgdk/bgdk/kustomization.yaml

Меняю цвет

<br/>

```
$ argocd app sync bgdk-app
```

<br/>

```
// [OK!] Цвет обновился!
http://192.168.49.2:31257
```

<br/>

```
$ argocd app delete bgdh-app
```
