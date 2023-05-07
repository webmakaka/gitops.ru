---
layout: page
title: Настроить использование DNS google на нодах kubernetes кластера
description: Настроить использование DNS google на нодах kubernetes кластера
keywords: devops, containers, kubernetes, dns
permalink: /devops/containers/kubernetes/kubeadm/dns/
---

# Настроить использование DNS google на нодах kubernetes кластера

<br/>

    $ kubectl edit cm -n kube-system coredns

<br/>

```
forward . /etc/resolv.conf
```

Меняем на

```
forward . 8.8.8.8:53
```

<br/>

Проверить, возможно можно подключившись к pod и выполнив внутри:

    nslookup kubernetes.default
