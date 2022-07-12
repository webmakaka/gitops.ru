---
layout: page
title: Yandex Clouds - Docker Registry
description: Yandex Clouds - Docker Registry
keywords: Deploy, Clouds, Yandex, Docker Registry
permalink: /tools/clouds/yandex/docker-registry/
---

# Yandex Clouds - Docker Registry

<br/>

```
// Создать реестр в Yandex Container Registry
$ yc container registry create --name my-registry
```

<br/>

```
Аутентифицируйтесь в Yandex Container Registr
$ yc container registry configure-docker
```

<br/>

```
$ cd ~/tmp/
$ vi Dockerfile
```

<br/>

```
FROM ubuntu:latest
RUN apt-get update -y
RUN apt-get install -y nginx
ENTRYPOINT ["nginx", "-g", "daemon off;"]
```

<br/>

```
$ YANDEX_REGISTRY_ID=<YOUR_YANDEX_REGISTRY_ID>
$ docker build . -t cr.yandex/${YANDEX_REGISTRY_ID}/ubuntu-nginx:latest
$ docker push cr.yandex/${YANDEX_REGISTRY_ID}/ubuntu-nginx:latest
```

<br/>

YANDEX CLOUD UI -> Container Registry -> ... -> ACL реестра

В списке ролей для allUsers уже отмечена роль viewer, отметьте вторую роль — container-registry.images.puller — и сохраните настройки.

<br/>

YANDEX CLOUD UI -> Compute Cloud -> Виртуальная машина -> Выбор образа/загрузочного диска -> Container Solution
