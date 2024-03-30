---
layout: page
title: 04. Deploy приложения с помощью GitLab и Helm в MiniKube
description: 04. Deploy приложения с помощью GitLab и Helm в MiniKube
keywords: devops, ci-cd, gitlab, kubernetes, docker, gitlab
permalink: /samples/ci-cd/gitlab/kubernetes/deploy-app-in-minikube-with-gitlab-and-helm/
---

# 04. Deploy приложения с помощью GitLab и Helm в MiniKube

<br/>

    $ cat ~/.kube/config | base64

<br/>

**GitLab -> Project -> Settings -> CI/CD -> Variables**

<br/>

Variables - не Protected

<br/>

KUBE_CONFIG -> результат выполнения cat.

<br/>

```
.gitlab-ci.yml
```

<br/>

```yaml
image: docker:stable

variables:
  DOCKER_TLS_CERTDIR: ''
  DOCKER_HOST: tcp://192.168.0.5:2375
  DOCKER_DRIVER: overlay2

services:
  - docker:stable-dind

before_script:
  - docker info
  - echo ${CI_REGISTRY}
  - echo ${REGISTRY_USER}
  - echo ${REGISTRY_PASSWORD} | docker login -u ${REGISTRY_USER} --password-stdin ${CI_REGISTRY}

stages:
  - build
  - release
  - deploy

backend-build:
  stage: build
  except:
    - tags
  script:
    - docker build ./apps/v1/app/backend/ -f ./apps/v1/app/backend/Dockerfile -t webmakaka/devops-backend:$CI_COMMIT_SHORT_SHA
    - docker push webmakaka/devops-backend:$CI_COMMIT_SHORT_SHA

backend-release:
  stage: release
  only:
    - tags
  script:
    - docker pull webmakaka/devops-backend:$CI_COMMIT_SHORT_SHA
    - docker tag webmakaka/devops-backend:$CI_COMMIT_SHORT_SHA webmakaka/devops-backend:$CI_COMMIT_TAG
    - docker push webmakaka/devops-backend:$CI_COMMIT_TAG

frontend-build:
  stage: build
  except:
    - tags
  script:
    - docker build ./apps/v1/app/frontend/ -f ./apps/v1/app/frontend/Dockerfile -t webmakaka/devops-frontend:$CI_COMMIT_SHORT_SHA
    - docker push webmakaka/devops-frontend:$CI_COMMIT_SHORT_SHA

frontend-release:
  stage: release
  only:
    - tags
  script:
    - docker pull webmakaka/devops-frontend:$CI_COMMIT_SHORT_SHA
    - docker tag webmakaka/devops-frontend:$CI_COMMIT_SHORT_SHA webmakaka/devops-frontend:$CI_COMMIT_TAG
    - docker push webmakaka/devops-frontend:$CI_COMMIT_TAG

deploy-app:
  stage: deploy
  script:
    - apk add -U openssl curl tar gzip bash ca-certificates git
    - curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && chmod +x kubectl && mv ./kubectl /usr/local/bin/
    - mkdir -p ~/.kube/ && echo $KUBE_CONFIG | base64 -d > ~/.kube/config
    - kubectl cluster-info
    - curl -L https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
    - helm upgrade myguestbook -i apps/v2/chart/guestbook
```

<br/>

Отработало норм.

<br/>

```
$ kubectl get pods
NAME                                    READY   STATUS    RESTARTS   AGE
myguestbook-backend-6b467894d6-kcz4p    1/1     Running   2          4m2s
myguestbook-database-684cfff8-gs66g     1/1     Running   0          4m2s
myguestbook-frontend-7f77554fc4-rc7pv   1/1     Running   0          4m2s
```

<br/>

Приложение запускается на:

frontend.minikube.local
