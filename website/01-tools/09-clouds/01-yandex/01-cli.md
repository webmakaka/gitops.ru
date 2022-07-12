---
layout: page
title: Yandex Clouds - CLI
description: Yandex Clouds - CLI
keywords: Deploy, Clouds, Yandex Clouds - CLI
permalink: /tools/clouds/yandex/cli/
---

# Yandex Clouds - CLI

Начало работы с интерфейсом командной строки

https://cloud.yandex.ru/docs/cli/quickstart

<br/>

### 01. Практическая работа. Создание виртуальных машин с помощью yc

<br/>

https://practicum.yandex.ru/trainer/ycloud/lesson/f5ef7735-66df-432c-ab99-2057874be107/

<br/>

**Создаем сеть**

```
$ yc vpc network create --name my-network
```

<br/>

**Создаем три подсети в разных зонах доступности**

<br/>

```
$ yc vpc subnet create --name my-subnet-1 \
  --zone ru-central1-a \
  --range 192.168.1.0/24 \
  --network-name my-network
```

<br/>

```
$ yc vpc subnet create --name my-subnet-2 \
  --zone ru-central1-b \
  --range 192.168.2.0/24 \
  --network-name my-network
```

<br/>

```
$ yc vpc subnet create --name my-subnet-3 \
  --zone ru-central1-c \
  --range 192.168.3.0/24 \
  --network-name my-network
```

<br/>

Создаем 3 ВМ в нужных зонах доступности и привязываем их к подсети.

<br/>

```
$ yc compute instance create --name my-instance-1 \
  --hostname my-instance-1 \
  --zone ru-central1-a \
  --create-boot-disk image-family=ubuntu-2004-lts,size=30,type=network-nvme \
  --image-folder-id standard-images \
  --memory 4 --cores 2 --core-fraction 100 \
  --network-interface subnet-name=my-subnet-1,nat-ip-version=ipv4 \
  --async
```

<br/>

```
$ yc compute instance create --name my-instance-2 \
  --hostname my-instance-2 \
  --zone ru-central1-b \
  --create-boot-disk image-family=ubuntu-2004-lts,size=30,type=network-nvme \
  --image-folder-id standard-images \
  --memory 4 --cores 2 --core-fraction 100 \
  --network-interface subnet-name=my-subnet-2,nat-ip-version=ipv4 \
  --async
```

<br/>

```
$ yc compute instance create --name my-instance-3 \
  --hostname my-instance-3 \
  --zone ru-central1-c \
  --create-boot-disk image-family=ubuntu-2004-lts,size=30,type=network-nvme \
  --image-folder-id standard-images \
  --memory 4 --cores 2 --core-fraction 100 \
  --network-interface subnet-name=my-subnet-3,nat-ip-version=ipv4 \
  --async
```

<br/>

```
// Проверка статуса операции
$ yc operation get c9q9v4bsn1hs9api4b1

// Подождать завершения операции
$ yc operation wait c9q9v4bsn1hs9api4b13
```

<br/>

```
// Убедитесь, что ВМ созданы
$ yc compute instance list
```

<br/>

```
+----------------------+---------------+---------------+---------+---------------+--------------+
|          ID          |     NAME      |    ZONE ID    | STATUS  |  EXTERNAL IP  | INTERNAL IP  |
+----------------------+---------------+---------------+---------+---------------+--------------+
| ef3f7j9c5eojq5adlb88 | my-instance-3 | ru-central1-c | RUNNING | 130.193.57.56 | 192.168.3.12 |
| epdut6e50qi40er9051f | my-instance-2 | ru-central1-b | RUNNING | 51.250.29.39  | 192.168.2.12 |
| fhmn7928s6uu2p0ds2am | my-instance-1 | ru-central1-a | RUNNING | 51.250.4.215  | 192.168.1.16 |
+----------------------+---------------+---------------+---------+---------------+--------------+
```

<br/>

```
// Вывод в json формате
$ yc compute instance list --format json
```

<br/>

Чтобы не тратить ресурсы облака понапрасну, в веб-консоли удалите три созданные ВМ: на следующих практических работах они вам не понадобятся.

<br/>

## 02. Практическая работа. Использование файлов спецификаций

<br/>

```
// Создать сервисный аккаунт yandex
$ yc iam service-account create --name my-robot \
    --description "this is my favorite service account"
```

<br/>

```
$ yc iam service-account list
```

<br/>

```
// DELETE
// $ yc iam service-account delete my-robot
```

<br/>

```
// Получить список стандартных image
$ yc compute image list --folder-id standard-images
```

<br/>

```
name: my-group
service_account_id: ajeu495h1s9tn1rorulb

instance_template:
    platform_id: standard-v1
    resources_spec:
        memory: 2g
        cores: 2
    boot_disk_spec:
        mode: READ_WRITE
        disk_spec:
            image_id: fd8fosbegvnhj5haiuoq
            type_id: network-hdd
            size: 32g
    network_interface_specs:
        - network_id: enpnr4onfs6ihtoao32u
          primary_v4_address_spec: { one_to_one_nat_spec: { ip_version: IPV4 }}
    scheduling_policy:
        preemptible: false
    metadata:
      user-data: |-
        #cloud-config
          package_update: true
          runcmd:
            - [ apt-get, install, -y, nginx ]
            - [/bin/bash, -c, 'source /etc/lsb-release; sed -i "s/Welcome to nginx/It is $(hostname) on $DISTRIB_DESCRIPTION/" /var/www/html/index.nginx-debian.html']

deploy_policy:
    max_unavailable: 1
    max_expansion: 0
scale_policy:
    fixed_scale:
        size: 3
allocation_policy:
    zones:
        - zone_id: ru-central1-a

load_balancer_spec:
    target_group_spec:
        name: my-target-group
```

<br/>

```
$ yc compute instance-group create --file specification.yaml
```

<br/>

```
$ yc compute instance-group list
```

<br/>

### Часть 2. Балансировщик

<br/>

```
$ yc load-balancer network-load-balancer create --region-id ru-central1 --name my-load-balancer --listener name=my-listener,external-ip-version=ipv4,port=80
```

<br/>

```
$ yc load-balancer target-group list
```

<br/>

```
yc load-balancer network-load-balancer attach-target-group b7r97ah2jn5rmo6k1dsk   --target-group target-group-id=b7r7cmdopr7bejtmj7dt,healthcheck-name=test-health-check,healthcheck-interval=2s,healthcheck-timeout=1s,healthcheck-unhealthythreshold=2,healthcheck-healthythreshold=2,healthcheck-http-port=80
```

<br/>

### Часть 3. Доступ к машинам группы

```
$ yc load-balancer network-load-balancer target-states b7rkqsnocbl7vgrbv6br --target-group-id b7r675m18nf06i36erd5
```

<br/>

### Часть 4. Обновление Instance Group

```
...
boot_disk_spec:
   mode: READ_WRITE
   disk_spec:
       image_id: fd87uq4tagjupcnm376a
       type_id: network-hdd
       size: 32g
...
```

<br/>

```
$ yc compute instance-group update --name my-group --file specification.yaml
```

```
...
deploy_policy:
    max_unavailable: 1
    max_expansion: 0
...
```

<br/>

Или

...
deploy_policy:
max_unavailable: 0
max_expansion: 1
...

<br/>

### Часть 5. Удаление машины из группы

```
$ yc compute instance delete fhmraoik1u9ur4mj0u6q
```

<br/>

### Часть 6. Удаление Instance Group

```
$ yc compute instance-group delete --name my-group

$ yc load-balancer network-load-balancer delete --name my-load-balancer
```
