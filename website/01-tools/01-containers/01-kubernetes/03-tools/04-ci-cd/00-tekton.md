---
layout: page
title: Инсталляция Tekton
description: Инсталляция Tekton
keywords: linux, kubernetes, Tekton
permalink: /tools/containers/kubernetes/tools/ci-cd/tekton/
---

# Tekton

<br/>

Делаю:  
25.05.2023

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

<br/>

#### Добавляем Tekton CRD в MiniKube

<br/>

```
$ kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
```

<br/>

```
$ tkn version
Client version: 0.31.0
Pipeline version: v0.48.0
```

<br/>

```
$ kubectl get pods -n tekton-pipelines
NAME                                           READY   STATUS    RESTARTS   AGE
tekton-events-controller-54854b8875-t4v88      1/1     Running   0          101s
tekton-pipelines-controller-58f8d4f964-6sm7z   1/1     Running   0          101s
tekton-pipelines-webhook-7db988ddc6-jfwn5      1/1     Running   0          98s
```

<br/>

```
$ kubectl get pods -n tekton-pipelines-resolvers
NAME                                                 READY   STATUS    RESTARTS   AGE
tekton-pipelines-remote-resolvers-85d9686f77-prnvh   1/1     Running   0          2m9s
```

<br/>

```
$ kubectl get crds
NAME                                       CREATED AT
clustertasks.tekton.dev                    2023-05-25T18:14:16Z
customruns.tekton.dev                      2023-05-25T18:14:16Z
pipelineruns.tekton.dev                    2023-05-25T18:14:17Z
pipelines.tekton.dev                       2023-05-25T18:14:16Z
resolutionrequests.resolution.tekton.dev   2023-05-25T18:14:17Z
taskruns.tekton.dev                        2023-05-25T18:14:17Z
tasks.tekton.dev                           2023-05-25T18:14:17Z
verificationpolicies.tekton.dev            2023-05-25T18:14:17Z
```

<br/>

#### Добавление Tekton Dashboard в MiniKube (Если нужно)

<br/>

```
$ kubectl apply --filename https://github.com/tektoncd/dashboard/releases/latest/download/tekton-dashboard-release.yaml
```

<br/>

**Подключиться к dashboard**

<br/>

```
$ kubectl --namespace tekton-pipelines port-forward svc/tekton-dashboard 8080:9097
```
