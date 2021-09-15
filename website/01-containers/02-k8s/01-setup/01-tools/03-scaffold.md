---
layout: page
title: Инсталляция scaffold в ubuntu 20.04
description: Инсталляция scaffold в ubuntu 20.04
keywords: gitops, containers, kubernetes, setup, tools, scaffold
permalink: /containers/k8s/setup/tools/scaffold/
---

# Инсталляция scaffold в ubuntu 20.04

Делаю:  
15.09.2021

<br/>

**scaffold - инструмент для разработки в kubernetes**

```
$ curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/latest/skaffold-linux-amd64

$ chmod +x skaffold
$ sudo mv skaffold /usr/local/bin

$ skaffold version
v1.31.0
```
