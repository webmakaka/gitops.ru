---
layout: page
title: Инсталляция Tekton в ubuntu 22.04
description: Инсталляция Tekton в ubuntu 22.04
keywords: tools, containers, kubernetes, ci-cd, tekton, инсталляция
permalink: /tools/containers/kubernetes/utils/ci-cd/tekton/
---

# Инсталляция Tekton в ubuntu 22.04

<br/>

**Делаю:**  
2024.03.08

<br/>

### Инсталляция Tekton CLI

<br/>

```
$ mkdir ~/tmp
$ cd ~/tmp/
```

<br/>

```
$ vi tekton-setup.sh
```

<br/>

```
#!/bin/bash

export LATEST_VERSION=$(curl --silent "https://api.github.com/repos/tektoncd/cli/releases/latest" | grep '"tag_name"' | sed -E 's/.*"([^"]+)".*/\1/')

export LATEST_VERSION_SHORT=$(curl --silent "https://api.github.com/repos/tektoncd/cli/releases/latest" | grep '"tag_name"' | sed -E 's/.*"([^"]+)".*/\1/' | cut -c 2-)

curl -LO "https://github.com/tektoncd/cli/releases/download/${LATEST_VERSION}/tkn_${LATEST_VERSION_SHORT}_$(uname -s)_$(uname -m).tar.gz"

sudo tar xvzf tkn_${LATEST_VERSION_SHORT}_$(uname -s)_$(uname -m).tar.gz -C /usr/local/bin/ tkn
```

<br/>

```
$ bash tekton-setup.sh
```

```
$ tkn version
Client version: 0.35.1
```

<br/>

### Добавляем Tekton CRD в MiniKube

<br/>

```
$ kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
```

<br/>

```
$ tkn version
Client version: 0.35.1
Pipeline version: v0.57.0
```

<br/>

```
$ kubectl get pods -n tekton-pipelines
NAME                                           READY   STATUS    RESTARTS   AGE
tekton-events-controller-77857f9b75-j26z2      1/1     Running   0          43s
tekton-pipelines-controller-6987c95899-5sxm5   1/1     Running   0          43s
tekton-pipelines-webhook-7f556bb7d9-kbn2t      1/1     Running   0          43s
```

<br/>

```
$ kubectl get pods -n tekton-pipelines-resolvers
NAME                                                READY   STATUS    RESTARTS   AGE
tekton-pipelines-remote-resolvers-f94cc8475-6dmnx   1/1     Running   0          59s
```

<br/>

```
$ kubectl get crds
NAME                                       CREATED AT
clustertasks.tekton.dev                    2024-03-08T10:02:41Z
customruns.tekton.dev                      2024-03-08T10:02:41Z
pipelineruns.tekton.dev                    2024-03-08T10:02:41Z
pipelines.tekton.dev                       2024-03-08T10:02:41Z
resolutionrequests.resolution.tekton.dev   2024-03-08T10:02:41Z
stepactions.tekton.dev                     2024-03-08T10:02:41Z
taskruns.tekton.dev                        2024-03-08T10:02:41Z
tasks.tekton.dev                           2024-03-08T10:02:41Z
verificationpolicies.tekton.dev            2024-03-08T10:02:41Z
```

<br/>

### Добавление Tekton Dashboard в MiniKube (Если нужно)

<br/>

```
$ kubectl apply --filename https://storage.googleapis.com/tekton-releases/dashboard/latest/release.yaml
```

<br/>

**Подключиться к dashboard**

<br/>

```
$ kubectl --namespace tekton-pipelines port-forward svc/tekton-dashboard 8080:9097
```

<br/>

```
$ localhost:8080
```

<br/>

### Installing Tekton Triggers (Если нужно)

<br/>

```
// Install the trigger custom resource definitions (CRDs)
$ kubectl apply -f https://storage.googleapis.com/tekton-releases/triggers/latest/release.yaml

// An interceptor is an object that contains the logic necessary to validate and filter webhooks coming from various sources.
$ kubectl apply -f https://storage.googleapis.com/tekton-releases/triggers/latest/interceptors.yaml
```

<br/>

```
// Examples
$ kubectl apply -f https://raw.githubusercontent.com/tektoncd/triggers/main/examples/rbac.yaml
```

<br/>

Now that Triggers is installed, you will be able to listen for events from GitHub, but for the webhooks to reach your cluster, you will need to expose a route to the outside world.

<br/>

```
$ kubectl get pods -n tekton-pipelines
NAME                                                READY   STATUS    RESTARTS   AGE
tekton-events-controller-77857f9b75-j26z2           1/1     Running   0          2m24s
tekton-pipelines-controller-6987c95899-5sxm5        1/1     Running   0          2m24s
tekton-pipelines-webhook-7f556bb7d9-kbn2t           1/1     Running   0          2m24s
tekton-triggers-controller-5b6d5f54b7-r5rvt         1/1     Running   0          36s
tekton-triggers-core-interceptors-f58696689-x8crk   1/1     Running   0          30s
tekton-triggers-webhook-689688fc54-d7964            1/1     Running   0          36s
```

<br/>

```
$ tkn version
Client version: 0.35.1
Pipeline version: v0.57.0
Triggers version: v0.26.1
Dashboard version: v0.44.0
```
