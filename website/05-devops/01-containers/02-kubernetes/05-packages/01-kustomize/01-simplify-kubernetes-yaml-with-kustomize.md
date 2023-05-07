---
layout: page
title: Simplify Kubernetes YAML with Kustomize
description: Simplify Kubernetes YAML with Kustomize
keywords: devops, linux, kubernetes, kustomize
permalink: /devops/containers/kubernetes/packages/kustomize/simplify-kubernetes-yaml-with-kustomize/
---

# Simplify Kubernetes YAML with Kustomize

<br/>

Делаю:  
11.02.2021

https://www.youtube.com/watch?v=5gsHYdiD6v8

<br/>

```
$ curl -s "https://raw.githubusercontent.com/\
kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash && chmod +x kustomize && sudo mv kustomize /usr/local/bin/
```

<br/>

```
$ cd ~/tmp
$ git clone https://github.com/marcel-dempers/docker-development-youtube-series
```

<br/>

**Дока:**

https://github.com/marcel-dempers/docker-development-youtube-series/tree/master/kubernetes/kustomize

<br/>

```
$ cd ~/tmp/docker-development-youtube-series/kubernetes/kustomize

$ kubectl kustomize ./application | kubectl apply -f -

```

<br/>

```
$ kubectl get pods --namespace example
NAME                              READY   STATUS              RESTARTS   AGE
example-deploy-6ff5bdf788-c7k2w   0/1     ContainerCreating   0          47s
example-deploy-6ff5bdf788-jvdqf   0/1     ContainerCreating   0          47s
```

<br/>

```
$ kubectl delete ns example
```

<br/>

### Overlays

<br/>

```
$ kubectl kustomize ./environments/development/ | kubectl apply -f -
```

<br/>

```
$ kubectl get pods --namespace example
NAME                              READY   STATUS              RESTARTS   AGE
example-deploy-6ff5bdf788-7g68f   0/1     ContainerCreating   0          3m50s
example-deploy-6ff5bdf788-llhcq   0/1     ContainerCreating   0          3m50s
example-deploy-6ff5bdf788-mpbcr   0/1     ContainerCreating   0          3m50s
example-deploy-6ff5bdf788-x72bd   0/1     ContainerCreating   0          3m50s

```

<br/>

```
$ kubectl delete ns example
```

<br/>

```
$ kubectl kustomize ./environments/production/ | kubectl apply -f -
```

<br/>

```
$ kubectl get pods --namespace example
NAME                             READY   STATUS    RESTARTS   AGE
example-deploy-7cbdc98cf-5tsc9   1/1     Running   0          62s
example-deploy-7cbdc98cf-ngh6m   1/1     Running   0          62s
```

<br/>

```
$ kubectl delete ns example
```
