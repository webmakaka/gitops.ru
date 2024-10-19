---
layout: page
title: Инсталляция и подготовка k3s для работы в ubuntu 22.04
description: Инсталляция и подготовка k3s для работы в ubuntu 22.04
keywords: ubuntu, containers, kubernetes, k3s, setup
permalink: /tools/containers/kubernetes/k3s/setup/
---

# Инсталляция и подготовка k3s для работы в ubuntu 22.04

<br/>

## Инсталляция k3s в ubuntu 22.04

<br/>

**Делаю:**  
2024.10.19

```
$ curl -sfL https://get.k3s.io | sh -
```

<br/>

```
$ sudo systemctl enable k3s.service
$ sudo systemctl start k3s.service
```

<br/>

```
System has not been booted with systemd as init system (PID 1). Can't operate.
Failed to connect to bus: Host is dow
```

<br/>

```
sudo k3s kubectl get node
```
