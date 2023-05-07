---
layout: page
title: Запуск GO программ в контейнере Docker
description: Запуск GO программ в контейнере Docker
keywords: Запуск GO программ в контейнере Docker
permalink: /dev/go/go-inside-docker/
---

# Запуск GO программ в контейнере Docker

    $ cd /tmp/

    $ git clone https://bitbucket.org/marley-golang/hw1_tree/
    $ cd hw1_tree/

    $ docker build -t mailgo_hw1 .

<br/>
    
Можно заменить команду в Dockerfile на:
    
<br/>
    
    $ go run main.go .
