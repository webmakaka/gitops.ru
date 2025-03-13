---
layout: page
title: Инсталляция kustomize в ubuntu 22.04
description: Инсталляция kustomize в ubuntu 22.04
keywords: gitops, containers, kubernetes, setup, kustomize
permalink: /tools/containers/kubernetes/utils/kustomize/
---

# Инсталляция kustomize в ubuntu 22.04

Date:  
2024.03.08

<br/>

```
$ curl -s "https://raw.githubusercontent.com/\
kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash && chmod +x kustomize && sudo mv kustomize /usr/local/bin/
```

<br/>

```
$ kustomize version
v5.3.0
```

<br/>

```
// Удалить при необходимости
// sudo rm /usr/local/bin/kustomize
```

<br/>

### Если нужна версия, например, 4.5.7

```
$ wget "https://raw.githubusercontent.com/\
kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"

$ chmod +x ./install_kustomize.sh

$ ./install_kustomize.sh 4.5.7

$ chmod +x kustomize && sudo mv kustomize /usr/local/bin/
```

<br/>

```
$ kustomize version
{Version:kustomize/v4.5.7 GitCommit:56d82a8378dfc8dc3b3b1085e5a6e67b82966bd7 BuildDate:2022-08-02T16:35:54Z GoOs:linux GoArch:amd64}
```
