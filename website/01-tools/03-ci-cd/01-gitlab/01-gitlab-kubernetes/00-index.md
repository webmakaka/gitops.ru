---
layout: page
title: Сборка и деплой js приложения из GitLab в Kubernetes
description: Сборка и деплой js приложения из GitLab в Kubernetes
keywords: devops, ci-cd, gitlab, kubernetes, docker
permalink: /tools/ci-cd/gitlab/kubernetes/
---

# Сборка и деплой js приложения из GitLab в Kubernetes

<br/>

(Предлагаю в качестве примера использовать следующее <a href="https://github.com/webmakaka/Packaging-Applications-with-Helm-for-Kubernetes">приложение</a> Angular + Node.js + MongoDB.

<br/>

### Инсталляция <a href="//sysadm.ru/devops/gitops/cvs/gitlab/setup/ubuntu/">GitLab</a>.

### Настройка <a href="//sysadm.ru/devops/gitops/cvs/gitlab/errors/">docker для запуска job'ов</a>.

<br/>

Клонируем приложение и пока работаем с контентом из каталога /apps/v1.

<br/>

**Kubernetes:**

Можно обойтись minikube (Что в принципе и происходит). С обычным kubernetes тоже работает. Но требуется больше ресурсов.

Но при желании, можно использовать скрипты для разварачивания локального kubernetes кластера, которые можно взять <a href="https://github.com/webmakaka/vagrant-kubernetes-3-node-cluster-ubuntu-20.04">здесь</a>.

<br/>

### [01. Сборка и push контейнеров в registry](/tools/ci-cd/gitlab/kubernetes/build-and-push/)

### [02. Запуск приложения в MiniKube с помощью Helm](/tools/ci-cd/gitlab/kubernetes/run-app-in-minikube/)

### [03. Настрока хоста с GitLab для работы с MiniKube](/tools/ci-cd/gitlab/kubernetes/prepare-gitlab-host-to-work-with-minikube/)

### [04. Deploy приложения с помощью GitLab и Helm в MiniKube](/tools/ci-cd/gitlab/kubernetes/deploy-app-in-minikube-with-gitlab-and-helm/)

### [05. Deploy приложения с помощью GitLab и Helm в MiniKube, обновляющегося при коммите или релизе](/tools/ci-cd/gitlab/kubernetes/deploy-app-in-minikube-with-gitlab-and-helm-with-updates-on-commit-or-release/)

### [06. Prometheus & Grafana](/tools/ci-cd/gitlab/kubernetes/prometheus-and-grafana/)

### [07. ELK & KIBANA](/tools/ci-cd/gitlab/kubernetes/elastic/)
