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
08.05.2023

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
Client version: 0.30.1
Pipeline version: v0.47.0
```

<br/>

```
$ watch kubectl get deployments,pods,services --namespace tekton-pipelines
NAME                                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/tekton-pipelines-controller   1/1     1            1           45s
deployment.apps/tekton-pipelines-webhook      1/1     1            1           44s

NAME                                               READY   STATUS    RESTARTS   AGE
pod/tekton-pipelines-controller-66754f98bb-mjbsw   1/1     Running   0          45s
pod/tekton-pipelines-webhook-5dd964c86c-2f7r9      1/1     Running   0          44s

NAME                                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                              AGE
service/tekton-pipelines-controller   ClusterIP   10.102.53.122   <none>        9090/TCP,8008/TCP,8080/TCP           45s
service/tekton-pipelines-webhook      ClusterIP   10.111.71.150   <none>        9090/TCP,8008/TCP,443/TCP,8080/TCP   44s
```

<br/>

```
$ kubectl get crds
NAME                                       CREATED AT
clustertasks.tekton.dev                    2023-05-08T15:34:44Z
customruns.tekton.dev                      2023-05-08T15:34:44Z
pipelineruns.tekton.dev                    2023-05-08T15:34:44Z
pipelines.tekton.dev                       2023-05-08T15:34:44Z
resolutionrequests.resolution.tekton.dev   2023-05-08T15:34:45Z
taskruns.tekton.dev                        2023-05-08T15:34:45Z
tasks.tekton.dev                           2023-05-08T15:34:45Z
verificationpolicies.tekton.dev            2023-05-08T15:34:45Z

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
