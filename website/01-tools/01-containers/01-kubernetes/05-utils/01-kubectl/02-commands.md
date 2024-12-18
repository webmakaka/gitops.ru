---
layout: page
title: Команды kubectl
description: Команды kubectl
keywords: gitops, containers, kubernetes, kubectl, commands
permalink: /tools/containers/kubernetes/utils/kubectl/commands/
---

# Команды kubectl

```
// Скачать лог

$ export NAME_SPACE=myspace
$ export POD=mypod

$ kubectl --namespace ${NAME_SPACE} logs $(kubectl get pods --namespace ${NAME_SPACE} -l "app=${POD}" -o jsonpath="{.items[0].metadata.name}") > ~/logs/${POD}.logs.txt
```

<br/>

```
// Скачать каталог
$ kubectl cp myns/mypod-with-id:/app ~/tmp/myappname/
```

<br/>

```
// Посмотреть image у pod
$ kubectl --kubeconfig ~/.kube/config_mynamespace -n mynamespace get pod podname-755f6ff87b-79vc6 -o jsonpath="{..image}"


// Тоже самое, но подлиннее

$ export KUBECONFIG=config_my
$ export NAME_SPACE=namespace_my
$ export POD=pod_my


$ kubectl --kubeconfig ~/.kube/${KUBECONFIG} --namespace ${NAME_SPACE} get pod $(kubectl --kubeconfig ~/.kube/${KUBECONFIG} get pods --namespace ${NAME_SPACE} -l "app=${POD}" -o jsonpath="{.items[0].metadata.name}") -o jsonpath="{..image}"
```
