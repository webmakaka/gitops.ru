---
layout: page
title: Docker в Linux
description: Docker в Linux
keywords: gitops, docker, Docker в Linux
permalink: /tools/containers/docker/
---

# Docker в Linux

<br/>

### Инсталляция Docker

[Инсталляция Docker и Docker-Compose](/tools/containers/docker/setup/)

<br/>

### Удаление созданных Docker ресурсов

<br/>

```
$ {
    docker system prune -af
    docker container prune -f
    docker image prune -f
    docker volume prune -f
    docker network prune -f
}
```
