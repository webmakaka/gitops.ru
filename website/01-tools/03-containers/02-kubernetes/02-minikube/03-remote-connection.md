---
layout: page
title: Удаленное подключение к хосту с minikube в ubuntu 20.04
description: Удаленное подключение к хосту с minikube в ubuntu 20.04
keywords: gitops, containers, kubernetes, setup, minikube, ubuntu, remote
permalink: /tools/containers/kubernetes/minikube/setup/remote-connection/
---

# Удаленное подключение к хосту с minikube в ubuntu 20.04

<br/>

**Делаю:**  
04.10.2021

<br/>

```
$ export \
  API_SERVER=192.168.0.11
```

<br/>

где 192.168.0.11 - адрес хоста на котором будет запущен minikube.

<br/>

Без apiserver-ips не будет работать proxy и будет ошибка "Unable to connect to the server: x509: certificate is valid for..."

<br/>

```
// Команда запуска minikube с apiserver-ips
$ minikube start --profile ${PROFILE} --embed-certs --apiserver-ips=${API_SERVER}
```

<br/>

```
$ minikube --profile ${PROFILE} ip
192.168.49.2
```

<br/>

```
$ kubectl cluster-info
```

<br/>

Сейчас в конфиге ~/.kube/config в ключе server прописано https://192.168.49.2:8443

<br/>

```
$ cat ~/.kube/config
server: https://192.168.49.2:8443
```

<br/>

### Устанавливаем Nginx

```
$ sudo apt update
$ sudo apt install -y nginx

$ cd /etc/nginx/
$ sudo cp nginx.conf nginx.conf.orig
```

<br/>

```
$ sudo vi nginx.conf
```

Добавил блок

```
stream {
  server {
      listen 192.168.0.11:51999;
      #TCP traffic will be forwarded to the specified server
      proxy_pass 192.168.49.2:8443;
  }
}
```

```
$ sudo nginx -t
$ sudo systemctl restart nginx
```

<br/>

```
$ vi ~/.kube/config
server: https://192.168.49.2:8443
```

меняю на

```
server: https://192.168.0.11:51999
```

<br/>

### Настраиваем Firewall

```
$ sudo ufw allow ssh
$ sudo ufw enable

$ sudo ufw allow from 192.168.0.11/24 to any port 51999

$ sudo ufw status verbose
```

<br/>

### Проверка

$ kubectl get pods
Unable to connect to the server: x509: certificate is valid for 192.168.49.2, 10.96.0.1, 127.0.0.1, 10.0.0.1, not 192.168.0.11

```
$ kubectl cluster-info
Kubernetes control plane is running at https://192.168.49.2:8443
CoreDNS is running at https://192.168.49.2:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

<br/>

### Проверка на удаленном хосте

<br/>

```
$ telnet 192.168.0.11 51999
```

<br/>

```
$ kubectl cluster-info
```

<br/>

### Дополнительно

**[Пример с драйвером kvm](https://www.zepworks.com/posts/access-minikube-remotely-kvm/)**

**[Пример с драйвером virtualbox](/samples/ci-cd/gitlab/kubernetes/prepare-gitlab-host-to-work-with-minikube/)**
