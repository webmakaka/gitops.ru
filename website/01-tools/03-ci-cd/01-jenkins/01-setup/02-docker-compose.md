---
layout: page
title: Запуск Jekins в docker-compose
description: Запуск Jekins в docker-compose
keywords: tools, ci-cd, jenkins, setup, docker-compose
permalink: /tools/ci-cd/jenkins/setup/docker-compose/
---

<br/>

# Запуск Jekins в docker-compose

<br/>

Делаю:  
2025.03.08

<br/>

Взято за основу [Cbtnuggets] [Knox Hutchinson] DevOps Tools Engineer (Exam 701-100) Online Training [ENG, 2024]

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
      dockerfile: jenkins.dockerfile
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

**jenkins.dockerfile**

```yaml
$ cat << EOF > jenkins.dockerfile
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

<br/>

**11. Continuous Delivery with Jenkins, Github, and K8s**

<br/>

```
$ mkdir -p /var/lib/jenkins/.kube/config
```

<br/>

Копируем kubeconfig -> /var/lib/jenkins/.kube/config

<br/>

```
$ sudo chmod 777 /var/lib/jenkins/.kube/
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
      dockerfile: jenkins.dockerfile
    ports:
      - '8080:8080'
    environment:
      - JAVA_OPTS=-Dhudson.security.csrf.GlobalCrumbIssuerConfiguration.DISABLE_CSRF_PROTECTION=true
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - jenkins_home:/var/jenkins_home
      - /var/lib/jenkins/.kube/config:/var/jenkins_home/.kube/config
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

**jenkins.dockerfile**

```yaml
$ cat << EOF > jenkins.dockerfile
FROM jenkins/jenkins:lts
USER root

# Install Docker in the Jenkins container
RUN apt-get update && \
    apt-get install -y docker.io && \
    apt-get clean

# Add Jenkins user to the Docker group and systemd-journal group
RUN usermod -aG docker,systemd-journal jenkins

RUN apt-get update && \
    apt-get install -y curl && \
    curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && \
    chmod +x kubectl && \
    mv kubectl /usr/local/bin/ && \
    apt-get clean

ENV KUBECONFIG=/var/jenkins_home/.kube/config

USER jenkins
EOF
```

<br/>

```
$ docker-compose up -d
```

<br/>

![Jenkins Pipeline](/img/tools/ci-cd/jenkins/setup/docker-compose/dockerhub1.png 'Jenkins Pipeline'){: .center-image }

<br/>

![Jenkins Pipeline](/img/tools/ci-cd/jenkins/setup/docker-compose/dockerhub2.png 'Jenkins Pipeline'){: .center-image }

```
Dashboard > Manage Jenkins > Credentials

System > Global credentials (unrestricted) > Add Credentials

Kind: Username with password

Scope: Global (Jenkins, nodes, items, all child items, etc)

Username: docker login

Password: docker token

ID: dockerhub-creds

Description: Docker hub creds for private repo access
```

<br/>

```
$ kubectl create secret docker-registry my-dockerhub-secret \
    --docker-username=${DOCKER_LOGIN} \
    --docker-password=${DOCKER_TOKEN} \
    --docker-email=${DOCKER_EMAIL}
```

<br/>

**mywebapp-deployment.yaml**

Положили в репо, рядом с jenkinsfile

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mywebapp-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mywebapp-deployment
  template:
    metadata:
      labels:
        app: mywebapp
    spec:
      containers:
        - name: mywebapp-container
          image: webmakaka/mywebapp-ci:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
        imagePullSecrets:
        - name: my-dockerhub-secret
```

<br/>

**mywebapp-service.yaml**

Положили в репо, рядом с jenkinsfile

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: mywebapp-service
spec:
  selector:
    app: mywebapp
  type: NodePort
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
      nodePort: 30080
EOF
```

<br/>

**jenkinsfile**

Не проверялось!!!

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
      stage('Build Final Production Image') {
        steps {
          script {
                sh "docker build -t wildmakaka/cats-app:latest ."
              }
            }
          }

      stage('Build Docker Image') {
        steps {
          script {
                // Log in to Docker Hub
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                  sh 'docker login -u $DOCKER_USER -p $DOCKER_PASS'

                }

                  // Tag the image
                  sh 'docker tag webmakaka/jenkinsci-demo:latest webmakaka/jenkinsci-demo:latest'

                  // Push the image to Docker Hub
                  sh 'docker push webmakaka/jenkinsci-demo:latest'
              }
            }
          }

      stage('Apply Deployment to Kubernetes') {
        steps {
          script {
                  // Push the image to Docker Hub
                  sh 'kubectl apply -f mywebapp-deployment.yaml'
                  sh 'kubectl apply -f mywebapp-service.yaml'
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
