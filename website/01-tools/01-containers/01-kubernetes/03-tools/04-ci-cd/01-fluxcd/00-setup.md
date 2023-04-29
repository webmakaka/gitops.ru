---
layout: page
title: Инсталляция fluxcd
description: Инсталляция fluxcd
keywords: linux, kubernetes, fluxcd, setup
permalink: /tools/containers/kubernetes/tools/ci-cd/fluxcd/setup/
---

# Инсталляция fluxcd

<br/>

Делаю:  
29.04.2023

<br/>

```
$ curl -s https://fluxcd.io/install.sh | sudo bash
```

<br/>

```
$ flux --version
flux version 2.0.0-rc.1
```

<br/>

Github -> Settings -> Developer Settings -> Personal acess tokens -> Tokens (classic) -> Generate new token (classic)

```
v repo
```

Generate Token

<br/>

```
// После развертывания предоставит данные
$ flux version
```
