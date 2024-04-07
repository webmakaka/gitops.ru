---
layout: page
title: GitOps Cookbook - Argo CD - Deploy from a Private Git Repository
description: GitOps Cookbook - Argo CD - Deploy from a Private Git Repository
keywords: books, gitops, argo-cd, Deploy from a Private Git Repository
permalink: /books/gitops/gitops-cookbook/argo-cd/deploy-from-a-private-git-repository/
---

<br/>

# [Book] [OK!] 7.6 Deploy from a Private Git Repository

<br/>

Делаю:  
2024.04.07

<br/>

1. Создаю private git repository https://github.com/wildmakaka/gitops-cookbook-sc-private.git

2. Добавляю в него https://github.com/gitops-cookbook/gitops-cookbook-sc.git

<br/>

```
$ argocd repo add git@github.com:wildmakaka/gitops-cookbook-sc-private.git \
  --ssh-private-key-path ~/.ssh/wildmakaka
```

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
    repoURL: git@github.com:wildmakaka/gitops-cookbook-sc-private.git
    path: ch07/bgd
    targetRevision: main
EOF
```

<br/>

```
$ argocd app list
```

<br/>

```
$ argocd app sync bgd-app
```

<br/>

```
$ kubectl get pods -n bgd
NAME                 READY   STATUS    RESTARTS   AGE
bgd-547cbdc7-ffcjh   1/1     Running   0          69s
```

<br/>

```
$ minikube --profile ${PROFILE} ip
192.168.49.2
```

<br/>

```
$ kubectl get services -n bgd
NAME   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
bgd    ClusterIP   10.100.85.97   <none>        8080/TCP   92s
```

<br/>

```
$ kubectl patch svc bgd -n bgd -p '{"spec": {"type": "NodePort"}}'
```

<br/>

```
$ kubectl get services -n bgd
NAME   TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
bgd    NodePort   10.100.85.97   <none>        8080:31050/TCP   111s
```

<br/>

```
// [OK!]
http://192.168.49.2:31050
```
