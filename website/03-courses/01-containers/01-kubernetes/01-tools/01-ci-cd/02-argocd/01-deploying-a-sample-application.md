---
layout: page
title: Introduction to Argo CD Kubernetes DevOps CI CD
description: Introduction to Argo CD Kubernetes DevOps CI CD
keywords: devops, contaiers, kubernetes, ci-cd, argocd
permalink: /courses/containers/kubernetes/ci-cd/argocd/introduction-to-argo-cd-kubernetes-devops-ci-cd/deploying-a-sample-application/
---

# Introduction to Argo CD Kubernetes DevOps CI/CD

<br/>

Делаю:  
09.05.2023

<br/>

## 04 - Argo CD Deep Dive

<br/>

### 12 - Deploying a sample application to Argo CD

<br/>

```
// $ argocd repo add "https://gitlab.com/abohmeed/samplegitopsapp.git" --username " [your username]" --password "[your personal token]"
```

<br/>

```
$ kubectl create ns microservices-demo
```

<br/>

```yaml
$ cat << EOF | kubectl apply -f -
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: microservices-demo
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/GoogleCloudPlatform/microservices-demo.git'
    path: kustomize
    targetRevision: main
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: microservices-demo
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
EOF
```

<br/>

```
$ argocd app sync microservices-demo
```

<br/>

```
$ kubectl get svc,pod -o wide -n microservices-demo
```

<br/>

```
$ kubectl -n microservices-demo port-forward svc/frontend-external 31801:80
```

```
// OK!
http://localhost:31801/
```
