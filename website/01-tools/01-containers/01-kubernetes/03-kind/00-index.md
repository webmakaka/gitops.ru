---
layout: page
title: Kind в linux
description: Kind в linux
keywords: gitops, containers, kubernetes, kind
permalink: /tools/containers/kubernetes/kind/
---

# Kind в linux

<br/>

**Делаю:**  
2025.04.27

<br/>

https://kind.sigs.k8s.io/docs/user/quick-start/#installing-from-release-binaries

<br/>

```
// Если ошибка
// curl: (35) error:0A00010B:SSL routines::wrong version number
// Качай по http, а не по https
$ [ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.27.0/kind-linux-amd64
$ chmod +x ./kind
$ sudo mv ./kind /usr/local/bin/kind
```

<br/>

```
$ kind --version
kind version 0.27.0
```

<br/>

### Download Cluster Configurations and Create a 3 Node Kubernetes Cluster as

```
$ cd ~/tmp
$ git clone https://github.com/initcron/k8s-code.git
$ cd k8s-code/helper/kind/
$ kind create cluster --config kind-three-node-cluster.yaml
```

<br/>

### Validate

```
$ kind get clusters
$ kubectl cluster-info --context kind-kind
```

<br/>

```
$ kubectl get nodes
NAME                 STATUS     ROLES           AGE   VERSION
kind-control-plane   NotReady   control-plane   20s   v1.32.2
kind-worker          NotReady   <none>          9s    v1.32.2
kind-worker2         NotReady   <none>          9s    v1.32.2
```

<br/>

### Restarting and Resetting the Cluster

```
// $ docker stop kind-control-plane kind-worker kind-worker2
// $ docker start kind-control-plane kind-worker kind-worker2
```

<br/>

```
$ kind get clusters
kind
```

<br/>

```
// Delete cluster
// $ kind delete cluster --name kind
```
