---
layout: page
title: Linkerd
description: Linkerd
keywords: devops, containers, kubernetes, service mesh, Linkerd
permalink: /tools/containers/kubernetes/utils/service-mesh/linkerd/setup/
---

# Инсталляция Linkerd

<br/>

```
$ curl -fsL https://run.linkerd.io/install | sh
```

<br/>

```
$ export PATH=$PATH:/home/a3333333/.linkerd2/bin
```

<br/>

```
$ linkerd check --pre
```

<br/>

```
$ linkerd install
```

<br/>

```
// ХЗ что делаем
$ linkerd install | kubectl apply --filename -
```

<br/>

```
$ linkerd check
```

<br/>

### Добавить дополнительные пакеты

```
// prometheus и grafana
$ linkerd viz install | kubectl apply --filename -
```

<br/>

```
// jaeger
$ linkerd jaeger install | kubectl apply --filename -
```

<br/>

```
$ linkerd check
```
