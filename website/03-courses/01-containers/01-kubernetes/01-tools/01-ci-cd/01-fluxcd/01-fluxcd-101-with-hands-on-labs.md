---
layout: page
title: FluxCD 101 with Hands-On Labs
description: FluxCD 101 with Hands-On Labs
keywords: linux, kubernetes, FluxCD
permalink: /courses/containers/kubernetes/ci-cd/fluxcd/fluxcd-101-with-hands-on-labs/
---

# [Video Course][Siddharth Barahalikar] FluxCD 101 with Hands-On Labs [ENG, 2023][~5h 45m]

### LAB 1 - Setup FluxCD Server and CLI

### LAB 2 - Deploy Application Manifest - Flux Repo

https://github.com/sid-demo?tab=repositories

clone -> bb-app-source к себе

https://github.com/sidd-harth/block-buster

https://github.com/sidd-harth-2

git clone https://github.com/sidd-harth-2/bb-app-source

git switch 1-demo

Скопировали манифесты в каталог 1-demo

в block-buseter/flux-cluters/dev-cluster/1-demo

commit / push

```
$ kubectl get ns
```

Должно появиться 1-demo

```
$ flux get sources git
```

```
$ flux get sources kustomizations
```

```
$ kubeclt -n 1-demo get all
```

localhost:30001
