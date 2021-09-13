---
layout: page
title: Jenkins
description: Jenkins
keywords: ci-cd, jenkins
permalink: /ci-cd/jenkins/
---

# Jenkins

Делаю:  
13.09.2021

<br/>

### Запуск Jenkins с помощью docker-compose

<br/>

    $ cd ~/projects/
    $ mkdir -p ./ci-cd/jenkins/jenkins_home
    $ cd ci-cd/jenkins/
    $ vi docker-compose.yml

<!--

    Можент быть нужно создать локального пользователя jenkins и назначить ему права.

    $ sudo chown -R 1000:1000 jenkins_home

-->

<br/>

**docker-compose.yml**

<br/>

```
version: '3'
services:
  jenkins:
    container_name: jenkins
    image: jenkins/jenkins
    ports:
      - "8080:8080"
    volumes:
      - $PWD/jenkins_home:/var/jenkins_home
    networks:
      - net
networks:
  net:
```

<br/>

    $ docker-compose up

<br/>

localhost:8080
