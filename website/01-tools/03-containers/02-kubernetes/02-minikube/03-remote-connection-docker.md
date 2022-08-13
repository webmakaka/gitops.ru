---
layout: page
title: Удаленное подключение к хосту с minikube в ubuntu 20.04 (Docker)
description: Удаленное подключение к хосту с minikube в ubuntu 20.04 (Docker)
keywords: gitops, containers, kubernetes, setup, minikube, ubuntu, remote
permalink: /tools/containers/kubernetes/minikube/setup/remote-connection-docker/
---

# Удаленное подключение к хосту с minikube в ubuntu 20.04 (Docker)

<br/>

**Делаю:**  
13.08.2022

<br/>

```
$ export \
  API_SERVER=192.168.1.101
```

<br/>

где 192.168.1.101 - адрес хоста на котором будет запущен minikube.

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
$ sudo vi /etc/nginx/nginx.conf
```

<br/>

Добавил блок

```
stream {
  server {
      listen 192.168.1.101:51999;
      #TCP traffic will be forwarded to the specified server
      proxy_pass 192.168.49.2:8443;
  }
}
```

<br/>

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
server: https://192.168.1.101:51999
```

<br/>

### Настраиваем Firewall

```
$ sudo ufw allow ssh
$ sudo ufw enable

$ sudo ufw allow from 192.168.1.101/24 to any port 51999

$ sudo ufw status verbose
```

<br/>

### Проверка на minikube хосте

<br/>

```
$ kubectl cluster-info
Kubernetes control plane is running at https://192.168.1.101:51999
CoreDNS is running at https://192.168.1.101:51999/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

<br/>

### Проверка на хосте с которого буду выполнять команды

<br/>

```
$ telnet 192.168.1.101 51999
```

<br/>

```
// Копирую ~/.kube/config с хоста с minikube на хост с которого буду выполнять команды.
$ scp 192.168.1.101:/home/marley/.kube/config ~/.kube/config
```

<br/>

```
$ kubectl cluster-info
Kubernetes control plane is running at https://192.168.1.101:51999
CoreDNS is running at https://192.168.1.101:51999/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

```

<br/>

```
$ kubectl get nodes
NAME              STATUS   ROLES           AGE   VERSION
marley-minikube   Ready    control-plane   24m   v1.24.3
```

<br/>

### Дополнительно

**[Еще 1 Пример с драйвером virtualbox](/tools/containers/kubernetes/minikube/setup/remote-connection-virtualbox/)**
