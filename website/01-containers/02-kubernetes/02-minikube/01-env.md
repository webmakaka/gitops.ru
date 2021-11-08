---
layout: page
title: Беслпатное облако Google для запуска примеров с kubernetes в minikube
description: Беслпатное облако Google для запуска примеров с kubernetes в minikube
keywords: gitops, containers, kubernetes, setup
permalink: /containers/kubernetes/google-cloud-shell/
---

# Беслпатное облако Google для запуска примеров с kubernetes в minikube

Нужно иметь гуглопочту.

Если 15-20 минут ничего не делать. Виртуалка удаляется.
Под домашний каталог дается что-то около 5GB. Эти данные остаются и не удаляются при удалении виртуалки.

4 ядра. 16 озу.

То, что перестартовывается, возможно, что даже к лучшему.

<br/>

### Подключение к бесплатному облаку от Google

<br/>

https://shell.cloud.google.com/

<br/>

**Инсталлим google-cloud-sdk**

https://cloud.google.com/sdk/docs/install

<br/>

```
$ gcloud auth login
$ gcloud cloud-shell ssh
```
