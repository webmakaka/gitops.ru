---
layout: page
title: Удаленное подключение к хосту с minikube в ubuntu 20.04 (VirtualBox)
description: Удаленное подключение к хосту с minikube в ubuntu 20.04 (VirtualBox)
keywords: gitops, containers, kubernetes, setup, minikube, ubuntu, remote, VirtualBox
permalink: /tools/containers/kubernetes/minikube/setup/remote-connection-virtualbox/
---

# Удаленное подключение к хосту с minikube в ubuntu 20.04 (VirtualBox)

<br/>

**Делаю:**  
12.08.2022

<br/>

```
$ export \
  API_SERVER=192.168.1.101
```

<br/>

где 192.168.1.101 - адрес хоста на котором будет запущен minikube.

<br/>

```
// Команда запуска minikube с apiserver-ips
$ minikube start --profile ${PROFILE} --embed-certs --apiserver-ips=${API_SERVER}
```

<br/>

Скопировал каталог .minikube на клиент. (Хотя, скорее всего нужно 2 или 3 сертификата).

<br/>

```
$ minikube stop --profile ${PROFILE}
```

<br/>

```
$ VBoxManage modifyvm ${vm} --natpf1 "kubectl,tcp,,51928,,8443"

$ VBoxManage showvminfo ${vm} --machinereadable | awk -F '[",]' '/^Forwarding/ { printf ("Rule %s host port %d forwards to guest port %d\n", $2, $5, $7); }'

Rule kubectl host port 51928 forwards to guest port 8443
Rule kubehttp80 host port 80 forwards to guest port 80
```

<!--

<br/>

```

$ VBoxManage modifyvm ${vm} --natpf1 "kubehttp8080,tcp,,8080,,8080"
$ VBoxManage modifyvm ${vm} --natpf1 "kubehttp80,tcp,,80,,80"

// Если нужно удалить
// $ VBoxManage modifyvm ${vm} --natpf1 delete kubehttp8080
// $ VBoxManage modifyvm ${vm} --natpf1 delete kubehttp80
```

-->

<br/>

```
$ minikube start --profile ${PROFILE}
```

<br/>

```
$ minikube --profile ${PROFILE} ip
192.168.59.107
```

<br/>

```
// C minikube хоста
// OK!
$ telnet 192.168.59.107 8443
```

<br/>

```
// C клинета
// OK!
$ telnet 192.168.1.101 51928
```

<br/>

// На клиенте

```
$ vi ~/.kube/config
```

<br/>

```yaml
apiVersion: v1
clusters:
  - cluster:
      insecure-skip-tls-verify: true
      server: https://192.168.1.101:51928
    name: minikube
contexts:
  - context:
      cluster: minikube
      user: minikube
    name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
  - name: minikube
    user:
      client-certificate: /home/marley/.minikube/profiles/marley-minikube/client.crt
      client-key: /home/marley/.minikube/profiles/marley-minikube/client.key
```

<br/>

Нужно поправить client-certificate и client-key

<br/>

```
$ kubectl get nodes
NAME              STATUS   ROLES           AGE   VERSION
marley-minikube   Ready    control-plane   50m   v1.24.3
```

<br/>

```
$ kubectl cluster-info
Kubernetes control plane is running at https://192.168.1.101:51928
CoreDNS is running at https://192.168.1.101:51928/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

<br/>

```
$ cd ~/tmp/
$ git clone https://github.com/webmakaka/cats-app.git
$ cd ~/tmp/cats-app/k8s/
```

<br/>

```
$ kubectl apply -f ./
```

<br/>

```
$ kubectl get pods
NAME                                            READY   STATUS    RESTARTS   AGE
minikube-cats-app-deployment-7b6d7d68fc-6dqgq   1/1     Running   0          59s
minikube-cats-app-deployment-7b6d7d68fc-fbvd4   1/1     Running   0          59s
minikube-cats-app-deployment-7b6d7d68fc-pdwll   1/1     Running   0          59s
```

<br/>

```
$ kubectl get ing
NAME              CLASS   HOSTS   ADDRESS          PORTS   AGE
ingress-service   nginx   *       192.168.59.107   80      77s
```

<br/>

```
// C minikube хоста
// OK!
$ curl 192.168.59.107
```

<br/>

```
// C клинета
// FAIL!
$ curl 192.168.1.101
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
      listen 192.168.1.101:8080;
      proxy_pass 192.168.59.107:80;
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
// OK!
http://192.168.1.101:8080/
```

<br/>

### Дополнительно

**[Пример с драйвером kvm](https://www.zepworks.com/posts/access-minikube-remotely-kvm/)**

**[Еще 1 Пример с драйвером virtualbox](/samples/ci-cd/gitlab/kubernetes/prepare-gitlab-host-to-work-with-minikube/)**
