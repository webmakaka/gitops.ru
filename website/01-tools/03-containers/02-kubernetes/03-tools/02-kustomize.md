---
layout: page
title: Инсталляция kustomize в ubuntu 20.04
description: Инсталляция kustomize в ubuntu 20.04
keywords: gitops, containers, kubernetes, setup, kustomize
permalink: /tools/containers/kubernetes/tools/kustomize/
---

# Инсталляция kustomize в ubuntu 20.04

Date:  
19.01.2023

<br/>

```
$ curl -s "https://raw.githubusercontent.com/\
kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash && chmod +x kustomize && sudo mv kustomize /usr/local/bin/
```

<br/>

```
$ kustomize version
{Version:kustomize/v4.5.7 GitCommit:56d82a8378dfc8dc3b3b1085e5a6e67b82966bd7 BuildDate:2022-08-02T16:35:54Z GoOs:linux GoArch:amd64}

```
