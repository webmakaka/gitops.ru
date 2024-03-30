---
layout: page
title: GitOps Cookbook - Argo CD - Automatic Synchronization
description: GitOps Cookbook - Argo CD - Automatic Synchronization
keywords: books, gitops, argo-cd, Automatic Synchronization
permalink: /books/gitops/gitops-cookbook/argo-cd/automatic-synchronization/
---

<br/>

# [Book] [OK!] GitOps Cookbook: 07. Argo CD: 7.2 Automatic Synchronization

<br/>

Делаю:  
2024.03.09

<br/>

```yaml
$ cat << 'EOF' | kubectl create -f -
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: bgd-app
  namespace: argocd
spec:
  destination:
    namespace: bgd
    server: https://kubernetes.default.svc
  project: default
  source:
    repoURL: https://github.com/wildmakaka/gitops-cookbook-sc.git
    path: ch07/bgd
    targetRevision: main
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
EOF
```

<br/>

```
$ minikube --profile ${PROFILE} ip
192.168.49.2
```

<br/>

```
$ kubectl patch svc bgd -n bgd -p '{"spec": {"type": "NodePort"}}'
```

<br/>

```
$ kubectl get services -n bgd
NAME   TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
bgd    NodePort   10.100.45.5   <none>        8080:30411/TCP   5m39s
```

<br/>

```
// [OK!]
http://192.168.49.2:31739
```

<br/>

```
$ kubectl -n bgd patch deploy/bgd \
--type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/env/0/value", "value":"red"}]'
```

<br/>

```
// [OK!]
http://192.168.49.2:31739
```

<br/>

```
$ argocd app delete bgd-app
```
