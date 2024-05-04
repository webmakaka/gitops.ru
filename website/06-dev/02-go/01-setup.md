---
layout: page
title: Подготовка окружения запуска приложений и программирования в Linux на Golang
description: Подготовка окружения запуска приложений и программирования в Linux на Golang
keywords: dev, golang, linux, setup
permalink: /dev/go/setup/
---

# Подготовка окружения запуска приложений и программирования в Linux на Golang

<br/>

### Мой вариант инсталляции GO (в каталог /opt)

<br/>

**Делаю:**  
2024.05.04

<br/>

```
$ mkdir ~/tmp
$ cd ~/tmp/
$ wget --no-check-certificate https://golang.org/dl/go1.22.2.linux-amd64.tar.gz
```

<br/>

```
$ tar -xvzpf go1.22.2.linux-amd64.tar.gz
$ sudo mkdir -p /opt/go.1.22
$ sudo mv go/* /opt/go.1.22/
$ sudo ln -s /opt/go.1.22/ /opt/go
```

<br/>

```
$ mkdir -p ~/projects/golang/
```

<br/>

```
$ sudo vi /etc/profile.d/golang.sh
```

<br/>

```
#### GO 1.22 ########################

export GO_HOME=/opt/go
export PATH=${GO_HOME}/bin:$PATH

export GOPATH=~/projects/golang/
export PATH=${GOPATH}/bin:$PATH

#### GO 1.22 ########################
```

<br/>

```
$ sudo chmod +x /etc/profile.d/golang.sh
$ source /etc/profile.d/golang.sh
```

<br/>

```
$ go version
go version go1.22.2 linux/amd64
```

<br/>

### Доп плагины для разработки на GO в Visual Studio Code

Rich Go Language support for Visual Studio

```
^P
> Go Install/Update Tools
```
