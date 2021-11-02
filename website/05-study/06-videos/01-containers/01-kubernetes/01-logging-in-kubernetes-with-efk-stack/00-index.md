---
layout: page
title: Logging in Kubernetes with EFK Stack | The Complete Guide
description: Logging in Kubernetes with EFK Stack | The Complete Guide
keywords: containers, elastic, fluentd, kibana
permalink: /study/videos/containers/kubernetes/logging-in-kubernetes-with-efk-stack/
---

# [Nana Janashia] Logging in Kubernetes with EFK Stack | The Complete Guide [ENG, 2021]

<br/>

**node app:**  
https://gitlab.com/nanuchi/node-app

<br/>

**java app**
https://gitlab.com/nanuchi/java-app

<br/>

**Промо на облачный kubernetes от linode на $100**  
https://gitlab.com/nanuchi/efk-course-commands/-/tree/master

<br/>

```
$ docker login
```

<br/>

**Приватные репо не нужны:**

<br/>

**node-app**

```
$ cd ~/tmp
$ git clone https://gitlab.com/nanuchi/node-app.git
$ cd node-app
$ docker build -t node-app .
$ docker tag node-app webmakaka/node-1.0:latest
$ docker push webmakaka/node-1.0
```

<br/>

**java-app**

```
$ cd ~/tmp
$ git clone https://gitlab.com/nanuchi/java-app.git
$ cd java-app
$ ./gradlew build
$ docker build -t java-app .
$ docker tag java-app webmakaka/java-1.0:latest
$ docker push webmakaka/java-1.0
```
