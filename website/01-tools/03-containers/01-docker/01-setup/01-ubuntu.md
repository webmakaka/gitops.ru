---
layout: page
title: Инсталляция и Upgrade Docker в Ubuntu 20.04
description: Инсталляция и Upgrade Docker в Ubuntu 20.04
keywords: gitops, docker, docker-compose, инсталляция, linux, ubuntu, bash скрипт
permalink: /tools/containers/docker/setup/ubuntu/
---

# Инсталляция / Upgrade Docker в Ubuntu 20.04

Делаю:  
07.03.2022

<br/>

### Инсталляция Docker версии 19.x

<br/>

```
$ mkdir ~/tmp
$ cd ~/tmp
```

<br/>

```
$ vi install-docker-and-docker-compose.sh
```

<br/>

```
#!/bin/bash

### Install Docker

sudo apt-get update
sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt-get update
sudo apt-get install -y docker-ce


### Install Docker-Compose

LATEST_VERSION=$(curl --silent "https://api.github.com/repos/docker/compose/releases/latest" | grep '"tag_name"' | sed -E 's/.*"([^"]+)".*/\1/')

sudo curl -L "https://github.com/docker/compose/releases/download/${LATEST_VERSION}/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

<br/>

```
$ chmod +x ./install-docker-and-docker-compose.sh
$ sudo ./install-docker-and-docker-compose.sh
```

<br/>

```
$ docker -v
Docker version 20.10.12, build e91ed57

$ docker-compose --version
Docker Compose version v2.3.0
```

<br/>

### Предоставить пользователю права для работы с docker

    // Добавить текущего пользоателя в группу для работы с docker
    $ sudo usermod -aG docker ${USER}

<br/>

в группе docker должен появиться этот пользователь

```
$ cat /etc/group | grep docker
docker:x:126:username
```

<br/>

Перелогиниваемся, иначе не будет работать

    $ logout

Лучше даже сделать reboot.

    $ sudo reboot

<br/>

### (При необходимости!) Изменить каталог по умолчанию для хранения контейнеров и имиджей

<br/>

    # mkdir -p /mnt/dsk1/docker
    # chown -R <username> /mnt/dsk1/docker

    # vi /etc/default/docker

    DOCKER_OPTS="-g /mnt/dsk1/docker"

<br/>

    # service docker restart

<br/>

    # ps auxwww | grep docker
    root      2476  0.0  0.1 274324 29896 ?        Ssl  10:10   0:00 /usr/bin/docker daemon -g /mnt/dsk1/docker

<br/>

### Разрешить работы с определенными registry по HTTP

<br/>

```
$ sudo vi /etc/docker/daemon.json
```

<br/>

```
{
    "insecure-registries": ["localhost:5000"]
}
```

<br/>

```
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

<br/>

```
$ docker info
***
Insecure Registries:
localhost:5000
127.0.0.0/8
```
