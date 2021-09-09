---
layout: page
title: 03. Настрока хоста с GitLab для работы с MiniKube
description: 03. Настрока хоста с GitLab для работы с MiniKube
keywords: devops, ci-cd, gitlab, kubernetes, docker, run app in minikube with helm
permalink: /ci-cd/gitlab-kubernetes/prepare-gitlab-host-to-work-with-minikube/
---

# 03. Настрока хоста с GitLab для работы с MiniKube

<br/>

<!--

$ minikube --profile devops-app start --apiserver-ips=192.168.0.5

-->

### На хост машине

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

В интернетах инструкции, в которых предлагают делать PORT FORWARD для 8443. У меня сам форвард не заработал.

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

```
$ kubectl get nodes
NAME         STATUS   ROLES                  AGE   VERSION
devops-app   Ready    control-plane,master   84m   v1.20.2
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

OK

<!--

<br/>

### Инструкции

https://www.systemcodegeeks.com/devops/remote-access-to-minikube-with-kubectl/

https://dzone.com/articles/access-minikube-using-kubectl-from-remote-machine

<br/>

### Проверки

```
$ minikube --profile devops-app ip
$ export MINIKUBE_IP=192.168.99.100
$ ssh -i ~/.minikube/machines/devops-app/id_rsa docker@${MINIKUBE_IP}


$ telnet localhost 8443
Connected to localhost
```

<br/>

Извне тоже OK

```
$ telnet 192.168.99.100 8443
```

<br/>

```
$ telnet 192.168.99.100 51928
Trying 192.168.99.100...
telnet: Unable to connect to remote host: Connection refused
```

-->
