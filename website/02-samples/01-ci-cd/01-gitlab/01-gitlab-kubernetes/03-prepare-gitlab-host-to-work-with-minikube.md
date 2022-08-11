---
layout: page
title: 03. Настрока хоста с GitLab для работы с MiniKube
description: 03. Настрока хоста с GitLab для работы с MiniKube
keywords: devops, ci-cd, gitlab, kubernetes, docker, run app in minikube with helm
permalink: /samples/ci-cd/gitlab/kubernetes/prepare-gitlab-host-to-work-with-minikube/
---

# 03. Настрока хоста с GitLab для работы с MiniKube

<br/>

UPD. Запутался! Не смог повторить стустя какое-то время! (А, здесь драйвер используется vm-driver virtualbox. Это определенно все меняет!)

<br/>

**[Upd: Есть мысль, что можно сделать проще с использованием nginx](/tools/containers/kubernetes/minikube/setup/remote-connection/)**

<br/>

<!--
$ minikube --profile devops-app start --apiserver-ips=192.168.0.5
-->

<!--

    // root password: kubeadmin
    $ scp 192.168.1.101:/home/marley/.kube/config ~/.kube/config

-->

<br/>

### С машины где установлен и запущен minikube

```
$ cd ~/.minikube/

$ ls
addons	ca.pem	  files		     machines
cache	cert.pem  key.pem	     profiles
ca.crt	certs	  last_update_check  proxy-client-ca.crt
ca.key	config	  logs		     proxy-client-ca.key
```

<br/>

Нужно скопировать на виртуалку с gitlab:

<br/>

```
.minikube/ca.crt
.minikube/profiles/devops-app/client.crt
.minikube/profiles/devops-app/client.key
```

<br/>

### В виртуальной машине с GitLab

<!--

В интернетах инструкции, в которых предлагают делать PORT FORWARD для 8443. У меня сам форвард не заработал.

-->

<br/>

```
$ vi /home/gitlab/.kube/config
```

<br/>

```yaml
apiVersion: v1
clusters:
  - cluster:
      insecure-skip-tls-verify: true
      server: https://192.168.99.100:8443
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
      client-certificate: /home/gitlab/.minikube/profiles/devops-app/client.crt
      client-key: /home/gitlab/.minikube/profiles/devops-app/client.key
```

<br/>

Нужно заменить ссылки на сертификаты client-certificate и client-key на содержимое данных файлов. Для этого.

<br/>

```
$ kubectl config view --flatten=true  > /home/gitlab/.kube/config.txt
$ cp /home/gitlab/.kube/config.txt /home/gitlab/.kube/config
```

<br/>

```
$ kubectl get nodes
NAME         STATUS   ROLES                  AGE   VERSION
devops-app   Ready    control-plane,master   84m   v1.20.2
```

OK!

<br/>

### Инструкции

https://www.systemcodegeeks.com/devops/remote-access-to-minikube-with-kubectl/

https://dzone.com/articles/access-minikube-using-kubectl-from-remote-machine
