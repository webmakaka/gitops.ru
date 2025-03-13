---
layout: page
title: Jenkins
description: Jenkins
keywords: Jenkins, cicd, инсталляция, linux
permalink: /tools/ci-cd/jenkins/setup/linux/2/
---

# Jenkins

<br/>

### [Инсталляция Jenkins в ubuntu 18.04 из пакетов](//javadev.org/devtools/cicd/jenkins/setup/ubuntu/20.04/)

<br/>

### Запуск в docker контейнере

**jenkins.sh**

```
#!/bin/bash
docker run -p 8080:8080 -p 50000:50000 jenkins/jenkins
```

<br/>

```
// Получить пароль
$ docker exec <container_id> cat /var/jenkins_home/secrets/initialAdminPassword
```

<br/>

### [Примеры использования](https://github.com/webmakaka/Learn-DevOps-CI-CD-with-Jenkins-using-Pipelines-and-Docker)
