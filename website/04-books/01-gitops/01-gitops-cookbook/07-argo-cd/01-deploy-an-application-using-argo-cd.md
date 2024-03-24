---
layout: page
title: GitOps Cookbook - Argo CD - Deploy an Application Using Argo CD
description: GitOps Cookbook - Argo CD - Deploy an Application Using Argo CD
keywords: books, gitops, argo-cd, Deploy an Application Using Argo CD
permalink: /books/gitops/gitops-cookbook/argo-cd/deploy-an-application-using-argo-cd/
---

<br/>

# [Book] [OK!] GitOps Cookbook: 07. Argo CD: 7.1 Deploy an Application Using Argo CD

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
EOF
```

<br/>

```
$ argocd app list
NAME            CLUSTER                         NAMESPACE  PROJECT  STATUS     HEALTH   SYNCPOLICY  CONDITIONS  REPO                                                  PATH      TARGET
argocd/bgd-app  https://kubernetes.default.svc  bgd        default  OutOfSync  Missing  <none>      <none>      https://github.com/wildmakaka/gitops-cookbook-sc.git  ch07/bgd  main

```

<br/>

```
$ argocd app sync bgd-app
TIMESTAMP                  GROUP        KIND   NAMESPACE                  NAME    STATUS    HEALTH        HOOK  MESSAGE
2024-03-09T09:34:05+03:00   apps  Deployment         bgd                   bgd  OutOfSync  Missing
2024-03-09T09:34:05+03:00          Namespace                               bgd  OutOfSync  Missing
2024-03-09T09:34:05+03:00            Service         bgd                   bgd  OutOfSync  Missing
2024-03-09T09:34:05+03:00          Namespace                               bgd    Synced  Missing
2024-03-09T09:34:05+03:00          Namespace         bgd                   bgd   Running    Synced              namespace/bgd created
2024-03-09T09:34:05+03:00            Service         bgd                   bgd  OutOfSync  Missing              service/bgd created
2024-03-09T09:34:05+03:00   apps  Deployment         bgd                   bgd  OutOfSync  Missing              deployment.apps/bgd created
2024-03-09T09:34:05+03:00            Service         bgd                   bgd    Synced  Healthy              service/bgd created

Name:               argocd/bgd-app
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          bgd
URL:                https://192.168.49.2:32763/applications/bgd-app
Repo:               https://github.com/wildmakaka/gitops-cookbook-sc.git
Target:             main
Path:               ch07/bgd
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        Synced to main (3e63da1)
Health Status:      Progressing

Operation:          Sync
Sync Revision:      3e63da1e346378754db5c071be1604fcdb64c499
Phase:              Succeeded
Start:              2024-03-09 09:34:05 +0300 MSK
Finished:           2024-03-09 09:34:05 +0300 MSK
Duration:           0s
Message:            successfully synced (all tasks run)

GROUP  KIND        NAMESPACE  NAME  STATUS   HEALTH       HOOK  MESSAGE
       Namespace   bgd        bgd   Running  Synced             namespace/bgd created
       Service     bgd        bgd   Synced   Healthy            service/bgd created
apps   Deployment  bgd        bgd   Synced   Progressing        deployment.apps/bgd created
       Namespace              bgd   Synced
```

<br/>

```
$ kubectl get pods -n bgd
NAME                 READY   STATUS    RESTARTS   AGE
bgd-547cbdc7-6twpw   1/1     Running   0          29s
```

<br/>

```
$ minikube --profile ${PROFILE} ip
192.168.49.2
```

<br/>

```
$ kubectl get services -n bgd
NAME   TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
bgd    ClusterIP   10.100.45.5   <none>        8080/TCP   2m15s
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
http://192.168.49.2:30411
```

<br/>

# Обновление

<br/>

https://github.com/wildmakaka/gitops-cookbook-sc/blob/main/ch07/bgd/bgd-deployment.yaml

Меняю:

value: "blue" на "orange"

в

```yaml
spec:
  containers:
    - image: quay.io/rhdevelopers/bgd:1.0.0
      name: bgd
      env:
        - name: COLOR
          value: 'blue'
      resources: {}
```

<br/>

```
$ argocd app sync bgd-app
```

<br/>

```
// [OK!]
http://192.168.49.2:30411
```

<br/>

```
$ argocd app delete bgd-app
```
