---
layout: page
title: 05. Deploy приложения с помощью GitLab и Helm в MiniKube, обновляющегося при коммите или релизе
description: 05. Deploy приложения с помощью GitLab и Helm в MiniKube, обновляющегося при коммите или релизе
keywords: devops, ci-cd, gitlab, kubernetes, docker, helm, templates, minikube
permalink: /samples/ci-cd/gitlab/kubernetes/deploy-app-in-minikube-with-gitlab-and-helm-with-updates-on-commit-or-release/
---

# 05. Deploy приложения с помощью GitLab и Helm в MiniKube, обновляющегося при коммите или релизе

<br/>

### Устанавливаем динамические значения переменных в HelmChart

<br/>

apps/v1/chart/guestbook/charts/frontend/templates/frontend.yaml

<br/>

Прописываю:

```yaml
{% raw %}
image: webmakaka/devops-frontend:{{ .Values.image.tag }}
{% endraw %}
```

<br/>

apps/v1/chart/guestbook/charts/backend/templates/backend.yaml

<br/>

Прописываю:

```yaml
{% raw %}
image: webmakaka/devops-backend:{{ .Values.image.tag }}
{% endraw %}
```

<br/>

### Создаю файлы values.yaml

tag: "2.0" - значение по умолчанию.

<br/>

**chart/guestbook/charts/frontend/values.yaml**

```yaml
image:
  repository: webmakaka/devops-frontend
  tag: '2.0'
```

<br/>

**chart/guestbook/charts/backend/values.yaml**

```yaml
image:
  repository: webmakaka/devops-backend
  tag: '2.0'
```

<br/>

### Проверка генерации правильного конфига

```
$ helm delete myguestbook
```

<br/>

```
$ export CI_COMMIT_TAG=15aee951
```

<br/>

```
$ helm upgrade myguestbook -i apps/v1/chart/guestbook --reuse-values --set-string frontend.image.tag=$CI_COMMIT_TAG --set-string backend.image.tag=$CI_COMMIT_TAG --dry-run --debug
```

<br/>

**Возвращает:**

```bash
***
USER-SUPPLIED VALUES:
backend:
  image:
    tag: 15aee951
frontend:
  image:
    tag: 15aee951

COMPUTED VALUES:
backend:
  global: {}
  image:
    repository: webmakaka/devops-backend
    tag: 15aee951
database:
  global: {}
frontend:
  global: {}
  image:
    repository: webmakaka/devops-frontend
    tag: 15aee951
***
```

<br/>

### Deploy с помощью GitLab в MiniKube

Наверное, можно сделать разные переменны $KUBE_CONFIG, указывающие на разные кластеры. Либо же, добавить названия dev, test, prod к сервисам и деплоить на 1 хост.

Пока буду делать deploy на 1 кластер. И каждый новый коммит будет обновлять версию проекта, заменяя в том числе релиз.

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

deploy-app-test:
  stage: deploy
  except:
    - tags
  script:
    - apk add -U openssl curl tar gzip bash ca-certificates git
    - curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && chmod +x kubectl && mv ./kubectl /usr/local/bin/
    - mkdir -p ~/.kube/ && echo $KUBE_CONFIG | base64 -d > ~/.kube/config
    - kubectl cluster-info
    - curl -L https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
    - helm upgrade myguestbook -i apps/v1/chart/guestbook --reuse-values --set-string frontend.image.tag=$CI_COMMIT_SHORT_SHA --set-string backend.image.tag=$CI_COMMIT_SHORT_SHA

deploy-app-prod:
  stage: deploy
  only:
    - tags
  script:
    - apk add -U openssl curl tar gzip bash ca-certificates git
    - curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && chmod +x kubectl && mv ./kubectl /usr/local/bin/
    - mkdir -p ~/.kube/ && echo $KUBE_CONFIG | base64 -d > ~/.kube/config
    - kubectl cluster-info
    - curl -L https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
    - helm upgrade myguestbook -i apps/v1/chart/guestbook --reuse-values --set-string frontend.image.tag=$CI_COMMIT_TAG --set-string backend.image.tag=$CI_COMMIT_TAG
```

<br/>

### Провека

```
$ kubectl get pods
NAME                       READY   STATUS    RESTARTS   AGE
backend-6dc6bb859f-vx9n7   1/1     Running   1          2m26s
frontend-66ddd4f57-fltvq   1/1     Running   0          2m26s
mongodb-746c86846c-rc8hw   1/1     Running   0          2m26s

```

<br/>

```
$ kubectl describe pod frontend-66ddd4f57-fltvq | grep Image
    Image:          webmakaka/devops-frontend:d04d29cf
    Image ID:       docker-pullable://webmakaka/devops-frontend@sha256:176c5f87fdd8e0c3f0ccf54f45490f16d3758a95bb3b51506ad07b58cf80e487

```

<br/>

**Создаю в web интерфейсе новый релиз**

(Можно и в командной строке)

<br/>

![GitOps](/img/samples/ci-cd/gitlab/kubernetes/pic-lecture03-pic01.png?raw=true)

<br/>

```
$ kubectl get pods
NAME                       READY   STATUS    RESTARTS   AGE
backend-867b9b9989-c98zf   1/1     Running   0          52s
frontend-6477dbbc5-zvv8z   1/1     Running   0          52s
mongodb-746c86846c-rc8hw   1/1     Running   0          5m53s
```

<br/>

```
$ kubectl describe pod frontend-6477dbbc5-zvv8z | grep Image
    Image:          webmakaka/devops-frontend:0.0.11
    Image ID:       docker-pullable://webmakaka/devops-frontend@sha256:176c5f87fdd8e0c3f0ccf54f45490f16d3758a95bb3b51506ad07b58cf80e487
```
