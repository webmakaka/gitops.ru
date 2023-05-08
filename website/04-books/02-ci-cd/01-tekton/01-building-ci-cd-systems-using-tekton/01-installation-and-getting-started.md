---
layout: page
title: Building CI/CD Systems Using Tekton - Installation and Getting Started
description: Building CI/CD Systems Using Tekton - Installation and Getting Started
keywords: books, ci-cd, tekton, Installation and Getting Started
permalink: /books/ci-cd/tekton/building-ci-cd-systems-using-tekton/installation-and-getting-started/
---

# Chapter 3. Installation and Getting Started

<br/>

Делаю:  
24.10.2021

<br/>

### Подключение к бесплатному облаку от Google

Описание [здесь](/tools/containers/kubernetes/google-cloud-shell/)

<br/>

```
$ gcloud cloud-shell ssh
```

<br/>

1. Инсталляция [MiniKube](/tools/containers/kubernetes/minikube/setup/) (Ingress и остальное можно не устанавливать)

**Испольновалась версия KUBERNETES_VERSION=v1.22.2**

2. Инсталляция [Kubectl](/tools/containers/kubernetes/tools/kubectl/)

<br/>

#### [Инсталляция Tekton CLI](/tools/containers/kubernetes/tools/ci-cd/tekton/)

<br/>

https://shell.cloud.google.com/

Вверху справа 3-й значок слева

Preview on port 8080

Открывается окно Tekton Dashboard
