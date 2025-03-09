---
layout: page
title: Запус Jekins в docker-compose
description: Запус Jekins в docker-compose
keywords: tools, ci-cd, jenkins, setup, docker-compose
permalink: /tools/ci-cd/jenkins/setup/docker-compose/
---

<br/>

# Запус Jekins в docker-compose

<br/>

Делаю:  
2025.03.08

<br/>

Взято на основе

<br/>

### [Cbtnuggets] [Knox Hutchinson] DevOps Tools Engineer (Exam 701-100) Online Training [ENG, 2024]

<br/>

**8. Jenkins - The Build Stage**

<!-- <br/>

```
$ sudo usermod -aG docker jenkins
``` -->

<br/>

```
$ mkdir -p ~/projects/docker/jenkins/
$ cd ~/projects/docker/jenkins/
```

<br/>

**docker-compose.yaml**

```yaml
$ cat << EOF > docker-compose.yaml
version: ‘3'
services:
  jenkins:
    container_name: jenkins
    build:
      context: .
      dockerfile: Dockerfile.jenkins
    ports:
      - '8080:8080'
    environment:
      - JAVA_OPTS=-Dhudson.security.csrf.GlobalCrumbIssuerConfiguration.DISABLE_CSRF_PROTECTION=true
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - jenkins_home:/var/jenkins_home
    networks:
      - proxy_net

  nginx-proxy-manager:
    container_name: nginx-proxy-manager
    image: jc21/nginx-proxy-manager:latest
    restart: unless-stopped
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    environment:
      DB_SQLITE_FILE: '/data/database.sqlite'
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
    networks:
      - proxy_net

volumes:
  jenkins_home:
  data:
  letsencrypt:

networks:
  proxy_net:
    driver: bridge
EOF
```

<br/>

```
// Валидация
$ docker-compose -f docker-compose.yaml config --quiet && printf "OK\n" || printf "ERROR\n"
```

<br/>

**Dockerfile.jenkins**

```yaml
$ cat << EOF > Dockerfile.jenkins
FROM jenkins/jenkins:lts
USER root

# Install Docker in the Jenkins container
RUN apt-get update && \
    apt-get install -y docker.io && \
    apt-get clean

# Add Jenkins user to the Docker group and systemd-journal group
RUN usermod -aG docker,systemd-journal jenkins

USER jenkins
EOF
```

<br/>

```
$ docker-compose build jenkins
$ docker-compose up -d
```

<br/>

```
// Получить пароль
$ docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

<br/>

```
// Jenkins
http://localhost:8080/
```

<br/>

```
// Nginx Proxy Manager
http://localhost:81/login

login: admin@example.com
password: changeme
```

<br/>

```
Available plugins ->

// Должны быть
- Git plugin
- Github plugin

// Добавить
- Docker Pipeline
```

<br/>

// Dashboard > Manage Jenkins > System > Jenkins URL
// Указать адрес для jenkins

<br/>

### GitHub

```
// No GitHub Apps
https://github.com/settings/apps


// New
https://github.com/settings/apps/new


GitHub App name: JenkinsCiLab1
Homepage URL: https://1fb5-34-90-164-212.ngrok-free.app/
Webhook URL: https://1fb5-34-90-164-212.ngrok-free.app/github-webhook/


Repository permissions


Actions -> Read-only
Metadata -> Read-only
Pull requests -> Read-only

+ Push + Pull
```

<br/>

Generate a private key

<br/>

Install Application

<br/>

### Jenkins

```
Dashboard > Manage Jenkins > Credentials

System > Global credentials (unrestricted) > Add Credentials

Kind -> Github App

Id -> GHA

App ID -> Github App ID

Key -> выполнить

```

<br/>

![Jenkins Credentials](/img/tools/ci-cd/jenkins/setup/docker-compose/credentials-01.png 'Jenkins Credentials'){: .center-image }

<br/>

![Jenkins Credentials](/img/tools/ci-cd/jenkins/setup/docker-compose/credentials-02.png 'Jenkins Credentials'){: .center-image }

<br/>

```
$ openssl pkcs8 -topk8 -inform PEM -outform PEM -in jenkinscilab1.2025-03-08.private-key.pem -out new-key.pem -nocrypt
```

Клонирую

```
$ git clone https://github.com/webmakaka/cats-app

Создаю приватное.
```

<br/>

Добавляю Jenkinsfile в приватное repo.

<br/>

**jenkinsfile**

```
pipeline {
    agent any
    stages {
      stage('Clone Repository') {
      steps {
        git branch: 'main',
        url: 'https://github.com/wildmakaka/cats-app.git',
        credentialsId: 'GHA'
        }
      }
    stage('Build Docker Image') {
    steps {
      script {
            sh "docker build -t wildmakaka/cats-app:${env.BUILD_ID} ."
          }
        }
      }
    }
    post {
      always {
        cleanWs()
      }
  }
}
```

<br/>

credentialsId - GHA

<br/>

### Jenkins

Dashboard > Manage Jenkins > Security

<br/>

![Jenkins Security](/img/tools/ci-cd/jenkins/setup/docker-compose/security.png 'Jenkins Security'){: .center-image }

<br/>

Dashboard > All > New Item

```
Name: DockerBuilder
Type: Pipeline
```

<br/>

![Jenkins Pipeline](/img/tools/ci-cd/jenkins/setup/docker-compose/pipeline.png 'Jenkins Pipeline'){: .center-image }

<br/>

```
Triggers
+ GitHub hook trigger for GITScm polling
```

<br/>

```
permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock
***
dial unix /var/run/docker.sock: connect: permission denied
```

<br/>

```
// Host
$ ls -al /var/run/docker.sock
srw-rw---- 1 root docker 0 Mar  8 17:33 /var/run/docker.sock
```

<br/>

```
$ docker exec -it jenkins bash
```

<br/>

```
// Container
$ ls -al /var/run/docker.sock
srw-rw---- 1 root 996 0 Mar  8 17:33 /var/run/docker.sock
```

<br/>

```
$ sudo usermod -aG docker $USER
$ sudo chown $USER /var/run/docker.sock
```
