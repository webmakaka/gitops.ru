---
layout: page
title: GitOps Cookbook - Argo CD - Image Updater
description: GitOps Cookbook - Argo CD - Image Updater
keywords: books, gitops, argo-cd, Image Updater
permalink: /books/gitops/gitops-cookbook/argo-cd/image-updater/
---

<br/>

# [Book] [OK!] 7.5 Image Updater

<br/>

Делаю:  
2024.03.25

<br/>

https://argocd-image-updater.readthedocs.io/en/stable/basics/update-methods/

<br/>

```
$ kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj-labs/argocd-image-updater/stable/manifests/install.yaml
```

<br/>

```
$ kubectl get pods -n argocd
```

<br/>

```
$ kubectl --namespace argocd create secret generic git-creds \
    --from-literal=username=<YOUR_GITHUB_USERNAME> \
    --from-literal=password=<YOUR_GITHUB_TOKEN>
```

<br/>

```
$ docker pull quay.io/rhdevelopers/bgd
```

<br/>

```
$ docker login
$ docker tag quay.io/rhdevelopers/bgd webmakaka/bgd:1.0.0
$ docker push webmakaka/bgd:1.0.0
```

<br/>

```
gitops-cookbook-sc/ch07/bgdui/base/bgd-deployment.yaml
```

<br/>

```yaml
containers:
  - image: webmakaka/bgd:1.0.0
```

<br/>

```
// Переименовываю
gitops-cookbook-sc/ch07/bgdui/bgdk/.argocd-source-bgdk-app.yaml
```

<br/>

```yaml
$ cat << 'EOF' | kubectl create -f -
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: bgdk-app
  namespace: argocd
  annotations:
    argocd-image-updater.argoproj.io/image-list: webmakaka/bgd
    argocd-image-updater.argoproj.io/write-back-method: git:secret:argocd/git-creds
    argocd-image-updater.argoproj.io/git-branch: main
spec:
  project: default
  source:
    repoURL: https://github.com/wildmakaka/gitops-cookbook-sc.git
    targetRevision: main
    path: ch07/bgdui/bgdk
  destination:
    server: https://kubernetes.default.svc
    namespace: bgdk
  syncPolicy:
      automated:
        selfHeal: true
        prune: true
        allowEmpty: true
EOF
```

<br/>

```
$ docker tag webmakaka/bgd:1.0.0 webmakaka/bgd:1.1.0
$ docker push webmakaka/bgd:1.1.0
```

<br/>

```
// Wait for 2 minutes
$ kubectl logs argocd-image-updater-59c45cbc5c-kjjtp -f -n argocd
```

<br/>

Образ обновился.  
Файл в репо сгенерился новый.

<br/>

### С приватным репо

Когда-то потом ....
