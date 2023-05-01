---
layout: page
title: Евгений Брикман -  Terraform. Инфраструктура на уровне кода [RUS, 2020]
description: Евгений Брикман - Terraform. Инфраструктура на уровне кода [RUS, 2020]
keywords: devops, tools, terraform, Terraform. Инфраструктура на уровне кода
permalink: /tools/terraform/terraform-up-and-running/
---

# [Евгений Брикман] Terraform. Инфраструктура на уровне кода [RUS, 2020]

Оригинал от 2019

github.com/brikis98/terraform-up-and-running-code

<br/>

AWS

<br/>

**Пользователю нужно добавить прав:**

<br/>

- AmazonEC2FullAccess
- AmazonS3FullAccess
- AmazonDynamoDBFullAccess
- AmazonRDSFullAccess
- CloudWatchFullAccess
- IAMFullAccess

<br/>

    $ cd ~/tmp
    $ git clone https://github.com/wildmakaka/terraform-up-and-running-code

<br/>

### Развертывание одного сервера

    $ cd terraform-up-and-running-code/code/tools/terraform/02-intro-to-terraform-syntax/one-server/

    $ terraform init

    $ terraform plan

    $ terraform apply

<br/>

### Развертывание одного веб-сервера

<br/>

    $ cd one-webserver

    $ terraform init
    $ terraform apply

    $ terraform output
    $ terraform output public_ip

    $ curl http://<PUBLIC_IP>:8080

<br/>

### Развертывание кластера веб-серверов и балансировщика нагрузки

    $ webserver-cluster

    $ terraform init
    $ terraform apply

    $ terraform output

    $ curl http://<alb_dns_name>:80

<br/>

### Удаление ненужных ресурсов

    $ terraform destroy
