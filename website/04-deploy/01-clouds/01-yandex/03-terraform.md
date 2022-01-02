---
layout: page
title: Yandex Clouds - Terraform
description: Yandex Clouds - Terraform
keywords: Deploy, Clouds, Yandex, Terraform
permalink: /deploy/clouds/yandex/terraform/
---

# Yandex Clouds - Terraform

Как использовать спецификации Terraform
Инфраструктура разворачивается в три этапа:

1. Команда terraform init инициализирует провайдеров, указанных в файле спецификации.

2. Команда terraform plan запускает проверку спецификации. Если есть ошибки — появятся предупреждения. Если ошибок нет, отобразится список элементов, которые будут созданы или удалены.

3. Команда terraform apply запускает развёртывание инфраструктуры.

Если инфраструктура не нужна, её можно уничтожить командой terraform destroy.

<br/>

# Практическая работа. Создаём виртуальную машину из образа и базу данных

https://learn.hashicorp.com/tutorials/terraform/install-cli

<br/>

```
$ cd ~/tmp
$ vi my-config.tf
```

<br/>

```
$ yc config list
$ yc vpc subnet list
```

<br/>

```
terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
}

provider "yandex" {
  token  =  "ваш OAuth токен"
  cloud_id  = "идентификатор облака"
  folder_id = "идентификатор каталога"
  zone      = "ru-central1-a"
}
```

<br/>

```
$ terraform init
```
