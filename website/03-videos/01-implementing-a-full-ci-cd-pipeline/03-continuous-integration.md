---
layout: page
title: Видеокурсы по DevOps - Внедрение полного конвейера CI/CD - Непрерывная интеграция
description: Видеокурсы по DevOps - Внедрение полного конвейера CI/CD - Непрерывная интеграция
keywords: Видеокурсы по DevOps, Внедрение полного конвейера CI/CD, Непрерывная интеграция
permalink: /videos/devops/implementing-a-full-ci-cd-pipeline/continuous-integration/
---

# [A Cloud Guru, Linux Academy] Внедрение полного конвейера CI/CD [RUS, 2020]

<br/>

## 04. Непрерывная интеграция

<br/>

### 15. Установка Jenkins

Устанавливаю <a href="//javadev.org/devtools/cicd/jenkins/setup/ubuntu/20.04/">Jenkins</a>

<br/>

### 16. Настройка проектов Jenkins

Клонируем к себе в репозиторий:

https://github.com/linuxacademy/cicd-pipeline-train-schedule-jenkins

<br/>

**Jenkins**

New Item

Name: train-schedule

Freestyle project

OK

<br/>

Source Code Management

Git -> https://github.com/linuxacademy/cicd-pipeline-train-schedule-jenkins

<br/>

Build -> Add build step -> Invoke Gradle script

Use Gradle Wrapper

Tasks -> build

<br/>

Post-build Actions -> Archive the artifacts

Files to archive -> dist/trainSchedule.zip

<br/>

Buil Now

<br/>

### 17. Запуск сборок с хуками в Git

**GitHub**

GitHub -> Settings -> Developer settings -> Personal access tokens -> Generate new token

<br/>

Token description: jenkins

-   admin:repo-hook

<br/>

Generate token

<br/>

Copy Api Key

<br/>

**Jenkins**

Manage Jenkins -> Configure System

Github -> GitHub Server

Name: GitHub

Credentials -> Add -> Jenkins

Kind: Secret Text

```
Secret: API_key
ID: gihtub_key
Description: GitHubKey
```

Add

<br/>

Credentials -> GitHubKey

-   Manage hooks

Save

<br/>

Выбираем созданные ранее проект train-schedule

Configure

Source Code Management

Git -> https://github.com/webmak1/cicd-pipeline-train-schedule-jenkins

<br/>

Build Trigger

-   GitHub hook trigger for GITScm polling

SAVE

<br/>

**GitHub**

Должен появиться Webhook.

App -> Settings -> Webhooks

Но он появится, только если использовать public ip.

С localhost будет вот такое:

<br/>

```
{"resource":"Hook","code":"custom","message":"Sorry, the URL host localhost is not supported because it isn't reachable over the public Internet"}
```

<br/>

Делаем коммит, изменив файл Readme.md

<br/>

Jenkins должен автоматически пересобрать проект.
