---
layout: page
title: Jenkins
description: Jenkins
keywords: ci-cd, jenkins
permalink: /tools/ci-cd/jenkins/setup/linux/
---

# Jenkins

Делаю:  
23.04.2023

<br/>

### Инсталляция в linux

https://www.jenkins.io/doc/book/installing/linux/#debianubuntu

<br/>

```
// Если нужно добавить пользователя
$ sudo usermod -aG docker jenkins
$ sudo systemctl restart jenkins
```

<br/>

### Запуск Jenkins в docker с помощью docker-compose

<br/>

```
$ cd ~/projects/
$ mkdir -p ./ci-cd/jenkins/
$ cd ci-cd/jenkins/

$ mkdir jenkins_home
$ sudo chown -R 1000:1000 jenkins_home
```

<br/>

**docker-compose.yml**

<br/>

```
$ vi docker-compose.yml
```

<br/>

```yaml
version: '3'
services:
  jenkins:
    container_name: jenkins
    image: jenkins/jenkins
    ports:
      - '8080:8080'
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

Устанавливаем plugin'ы

Админ не особо нужен.

<br/>

### Запуск примера с локальной установкой jenkins

С вариантом инсталляции с docker не пройдет, т.к. внутри нет docker.

<br/>

New Item ->

```
name: jenkins-pipeline
type: Pipeline
```

<br/>

Configure -> Pipeline ->

Definition: Pipeline script

Вставляем контент:

https://github.com/sandervanvugt/gitops/blob/main/jenkinspipe

Save -> Build Now

<br/>

### Указать правильную версию JDK

Скорее всего, нужно указать версию JDK 1.8

Нужна учетка на сайте oracle.com

<br/>

Manange Jenkins -> Global Tool Configuration -> JDK

Name: JDK8

Version 8u221

- I agree to the Java SE Development Kit License Agreement

<br/>

**Save**

<br/>

### [Learn DevOps: CI/CD with Jenkins using Pipelines and Docker](https://github.com/webmakaka/Learn-DevOps-CI-CD-with-Jenkins-using-Pipelines-and-Docker)
