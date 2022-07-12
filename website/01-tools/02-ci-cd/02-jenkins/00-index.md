---
layout: page
title: Jenkins
description: Jenkins
keywords: ci-cd, jenkins
permalink: /tools/ci-cd/jenkins/
---

# Jenkins

Делаю:  
13.09.2021

<br/>

### Запуск Jenkins с помощью docker-compose

<br/>

    $ cd ~/projects/
    $ mkdir -p ./ci-cd/jenkins/
    $ cd ci-cd/jenkins/

    $ mkdir jenkins_home
    $ sudo chown -R 1000:1000 jenkins_home

    $ vi docker-compose.yml

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

```
$ docker-compose up
```

<br/>

localhost:8080

<br/>

### Указать правильную версию jDK

Скорее всего, нужно указать версию JDK 1.8

Нужна учетка на сайте oracle.com

<br/>

Manange Jenkins -> Global Tool Configuration -> JDK

Name: JDK8

Version 8u221

- I agree to the Java SE Development Kit License Agreement

<br/>

**Save**
