---
layout: page
title: Clouds Amazon (AWS) - Kubernetes Setup (AWS)
description: Clouds Amazon (AWS) - Kubernetes Setup (AWS)
keywords: Clouds Amazon (AWS) - Kubernetes Setup (AWS)
permalink: /tools/clouds/aws/kubernetes/setup/manual/
---

# Clouds Amazon (AWS) - Kubernetes Setup (AWS)

<br/>

Делаю:  
09.10.2022

<br/>

**По материалам из курса The Ultimate Kubernetes Administrator Course**

Chapter 2: Build a K8s Cluster from Scratch

https://www.techworld-with-nana.com/kubernetes-administrator-cka

<br/>

**Имеем учебный аккаунт aws**

<br/>

![Clouds Amazon (AWS) - Kubernetes Setup (AWS)](/img/tools/clouds/aws/kubernetes/setup/manual/pic-01.png 'Clouds Amazon (AWS) - Kubernetes Setup (AWS)'){: .center-image }

<br/>

![Clouds Amazon (AWS) - Kubernetes Setup (AWS)](/img/tools/clouds/aws/kubernetes/setup/manual/pic-02.png 'Clouds Amazon (AWS) - Kubernetes Setup (AWS)'){: .center-image }

<br/>

По клику на open console открывается обычная консоль AWS с правами самого главного пользователя, который только может быть предоставлен пользовательской учетной записи в облках AWS.

<br/>

![Clouds Amazon (AWS) - Kubernetes Setup (AWS)](/img/tools/clouds/aws/kubernetes/setup/manual/pic-03.png 'Clouds Amazon (AWS) - Kubernetes Setup (AWS)'){: .center-image }

<br/>

### AIM

<br/>

**Создаем пользователя (admin), для того, чтобы не работать под главным пользователем.**

<br/>

![Clouds Amazon (AWS) - Kubernetes Setup (AWS)](/img/tools/clouds/aws/kubernetes/setup/manual/pic-04.png 'Clouds Amazon (AWS) - Kubernetes Setup (AWS)'){: .center-image }

<br/>

Users -> Add User

<br/>

![Clouds Amazon (AWS) - Kubernetes Setup (AWS)](/img/tools/clouds/aws/kubernetes/setup/manual/pic-05.png 'Clouds Amazon (AWS) - Kubernetes Setup (AWS)'){: .center-image }

<br/>

![Clouds Amazon (AWS) - Kubernetes Setup (AWS)](/img/tools/clouds/aws/kubernetes/setup/manual/pic-06.png 'Clouds Amazon (AWS) - Kubernetes Setup (AWS)'){: .center-image }

<br/>

![Clouds Amazon (AWS) - Kubernetes Setup (AWS)](/img/tools/clouds/aws/kubernetes/setup/manual/pic-07.png 'Clouds Amazon (AWS) - Kubernetes Setup (AWS)'){: .center-image }

<br/>

![Clouds Amazon (AWS) - Kubernetes Setup (AWS)](/img/tools/clouds/aws/kubernetes/setup/manual/pic-08.png 'Clouds Amazon (AWS) - Kubernetes Setup (AWS)'){: .center-image }

<br/>

![Clouds Amazon (AWS) - Kubernetes Setup (AWS)](/img/tools/clouds/aws/kubernetes/setup/manual/pic-09.png 'Clouds Amazon (AWS) - Kubernetes Setup (AWS)'){: .center-image }

<br/>

Записываем куда-нибудь.

- Username
- Secret access key
- Pass

Кликаем по ссылке вида:
https://069940897088.signin.aws.amazon.com/console

<br/>

![Clouds Amazon (AWS) - Kubernetes Setup (AWS)](/img/tools/clouds/aws/kubernetes/setup/manual/pic-10.png 'Clouds Amazon (AWS) - Kubernetes Setup (AWS)'){: .center-image }

<br/>

![Clouds Amazon (AWS) - Kubernetes Setup (AWS)](/img/tools/clouds/aws/kubernetes/setup/manual/pic-11.png 'Clouds Amazon (AWS) - Kubernetes Setup (AWS)'){: .center-image }

<br/>

### EC2

EC2 -> Instancec -> Launch an instance

<br/>

Создаем 3 виртуальные машины.

<br/>

```
1 master Ubuntu 2t.medium
2 worker Ubuntu 2t.large
```

![Clouds Amazon (AWS) - Kubernetes Setup (AWS)](/img/tools/clouds/aws/kubernetes/setup/manual/pic-12.png 'Clouds Amazon (AWS) - Kubernetes Setup (AWS)'){: .center-image }

<br/>

![Clouds Amazon (AWS) - Kubernetes Setup (AWS)](/img/tools/clouds/aws/kubernetes/setup/manual/pic-13.png 'Clouds Amazon (AWS) - Kubernetes Setup (AWS)'){: .center-image }

<br/>

![Clouds Amazon (AWS) - Kubernetes Setup (AWS)](/img/tools/clouds/aws/kubernetes/setup/manual/pic-14.png 'Clouds Amazon (AWS) - Kubernetes Setup (AWS)'){: .center-image }

<br/>

![Clouds Amazon (AWS) - Kubernetes Setup (AWS)](/img/tools/clouds/aws/kubernetes/setup/manual/pic-15.png 'Clouds Amazon (AWS) - Kubernetes Setup (AWS)'){: .center-image }

<br/>

![Clouds Amazon (AWS) - Kubernetes Setup (AWS)](/img/tools/clouds/aws/kubernetes/setup/manual/pic-16.png 'Clouds Amazon (AWS) - Kubernetes Setup (AWS)'){: .center-image }

<br/>

![Clouds Amazon (AWS) - Kubernetes Setup (AWS)](/img/tools/clouds/aws/kubernetes/setup/manual/pic-17.png 'Clouds Amazon (AWS) - Kubernetes Setup (AWS)'){: .center-image }

<br/>

![Clouds Amazon (AWS) - Kubernetes Setup (AWS)](/img/tools/clouds/aws/kubernetes/setup/manual/pic-18.png 'Clouds Amazon (AWS) - Kubernetes Setup (AWS)'){: .center-image }

<br/>

**Security -> Edit inbound rules**

```
// Порты берем здесь
https://kubernetes.io/docs/reference/ports-and-protocols/
```

172.31.0.0/16 - см. в VPC

![Clouds Amazon (AWS) - Kubernetes Setup (AWS)](/img/tools/clouds/aws/kubernetes/setup/manual/pic-19.png 'Clouds Amazon (AWS) - Kubernetes Setup (AWS)'){: .center-image }

<br/>

![Clouds Amazon (AWS) - Kubernetes Setup (AWS)](/img/tools/clouds/aws/kubernetes/setup/manual/pic-20.png 'Clouds Amazon (AWS) - Kubernetes Setup (AWS)'){: .center-image }

<br/>

![Clouds Amazon (AWS) - Kubernetes Setup (AWS)](/img/tools/clouds/aws/kubernetes/setup/manual/pic-21.png 'Clouds Amazon (AWS) - Kubernetes Setup (AWS)'){: .center-image }

<br/>

```
// + Для всех узлов weave
- 6783 Custom 172.31.0.0/16
```

![Clouds Amazon (AWS) - Kubernetes Setup (AWS)](/img/tools/clouds/aws/kubernetes/setup/manual/pic-22.png 'Clouds Amazon (AWS) - Kubernetes Setup (AWS)'){: .center-image }

<br/>

![Clouds Amazon (AWS) - Kubernetes Setup (AWS)](/img/tools/clouds/aws/kubernetes/setup/manual/pic-23.png 'Clouds Amazon (AWS) - Kubernetes Setup (AWS)'){: .center-image }

<br/>

![Clouds Amazon (AWS) - Kubernetes Setup (AWS)](/img/tools/clouds/aws/kubernetes/setup/manual/pic-24.png 'Clouds Amazon (AWS) - Kubernetes Setup (AWS)'){: .center-image }

<br/>

![Clouds Amazon (AWS) - Kubernetes Setup (AWS)](/img/tools/clouds/aws/kubernetes/setup/manual/pic-25.png 'Clouds Amazon (AWS) - Kubernetes Setup (AWS)'){: .center-image }

<br/>

![Clouds Amazon (AWS) - Kubernetes Setup (AWS)](/img/tools/clouds/aws/kubernetes/setup/manual/pic-26.png 'Clouds Amazon (AWS) - Kubernetes Setup (AWS)'){: .center-image }

<br/>

### [Host машина]

<br/>

```
$ cp ~/Downloads/k8s-node.pem ~/.ssh
$ chmod 400 ~/.ssh/k8s-node.pem
```

<br/>

### [Все узлы + Host машина] Прописываем узлы в /etc/hosts

<br/>

```
$ sudo vi /etc/hosts
```

<br/>

Заменить <Private_IP> на данные из консоли aws

<br/>

```
<Private_IP> master
<Private_IP> worker1
<Private_IP> worker2
```

<br/>

```
52.59.210.154 master
18.156.83.149 worker1
3.68.93.13 worker2
```

<br/>

**Подключение к узлам**

```
// master
$ ssh -i ~/.ssh/k8s-node.pem ubuntu@master

// workstation
$ ssh -i ~/.ssh/k8s-node.pem ubuntu@worker1
$ ssh -i ~/.ssh/k8s-node.pem ubuntu@worker2
```

<br/>

**Задание названий для узлов**

<br/>

```
$ sudo apt update -y && sudo apt upgrade -y
$ sudo swapoff -a

$ sudo hostnamectl set-hostname master
$ sudo hostnamectl set-hostname worker1
$ sudo hostnamectl set-hostname worker2
```

<br/>

### [Все узлы] Инсталляция container-d

<br/>

https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd

<br/>

```
$ mkdir ~/tmp
$ cd ~/tmp

$ vi install.sh
```

<br/>

```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system

sudo apt update
sudo apt install -y containerd

sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
systemctl status containerd
```

<br/>

```
$ chmod u+x install.sh
$ ./install.sh
```

<br/>

### [Все узлы] Install kubeadm kubelet and kubectl

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

<br/>

```
// Если нужно подобрать пакеты определенных версий
// $ apt-cache madison kubeadm
```

<br/>

```
$ vi install-k8s-components.sh
```

<br/>

```
sudo apt-get update

sudo apt-get install -y apt-transport-https ca-certificates curl

sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet=1.25.2-00 kubeadm=1.25.2-00 kubectl=1.25.2-00

# // фиксирует версии пакетов, чтобы они не обновлялись
sudo apt-mark hold kubelet kubeadm kubectl
```

<br/>

```
$ chmod +x install-k8s-components.sh
$ ./install-k8s-components.sh
```

<!-- <br/>

```
$ systemctl status kubelet
``` -->

<br/>

### [master]

```
$ sudo kubeadm init
```

<br/>

```
$ sudo kubectl get node --kubeconfig /etc/kubernetes/admin.conf
NAME STATUS ROLES AGE VERSION
master NotReady control-plane 20m v1.25.2
```

<br/>

```
$ mkdir -p ~/.kube
$ sudo cp -i /etc/kubernetes/admin.conf ~/.kube/config
$ sudo chown $(id -u):$(id -g) ~/.kube/config

$ ls -l ~/.kube/config
-rw------- 1 ubuntu ubuntu 5640 Oct 1 21:51 /home/ubuntu/.kube/config
```

<br/>

```
$ kubectl get node
NAME     STATUS     ROLES           AGE     VERSION
master   NotReady   control-plane   3m34s   v1.25.2

```

<br/>

### Установка и настройка драйвера сети weave

<br/>

https://www.weave.works/docs/net/latest/kubernetes/kube-addon/

<br/>

```
$ wget "https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml" -O weave.yaml
```

<br/>

```
vi weave.yaml
```

<br/>

```
          containers:
            - name: weave
              command:
                - /home/weave/launch.sh
```

<br/>

**Заменить на**

<br/>

```
          containers:
            - name: weave
              command:
                - /home/weave/launch.sh
                - --ipalloc-range=100.32.0.0/12
```

<br/>

```
$ kubectl apply -f weave.yaml
```

<br/>

```
$ kubectl get node
NAME     STATUS   ROLES           AGE   VERSION
master   Ready    control-plane   24m   v1.25.2
```

<br/>

```
$ kubectl get pod -n kube-system
```

coredns должны появиться

<br/>

### [Worker узлы] Подключение узлов к master

```
// master
// Если нужно заново запустить процедуру
$ kubeadm token create --print-join-command

// worker nodes
sudo kubeadm join \*\*\*
```

<br/>

```
// master
$ kubectl get node
NAME      STATUS     ROLES           AGE   VERSION
master    Ready      control-plane   53m   v1.25.2
worker1   Ready      <none>          86s   v1.25.2
worker2   NotReady   <none>          17s   v1.25.2
```

<!-- <br/>

```
$ kubectl get pod -n kube-syste -o wide | grep weave

$ kubectl exec -n kube-system weave-net-hxjdd -c weave -- /home/weave/weave --local status
``` -->

<!-- // local
$ scp -i ~/.ssh/k8s-node.pem ubuntu@master:/home/ubuntu/.kube/config ~/.kube/config -->

<br/>

### Test

```
$ git clone https://github.com/webmakaka/cats-app
$ cd cats-app/k8s/
$ kubectl apply -f ./
```

<br/>

### Перестартовка kubelet

```
// Перестартовать
$ sudo systemctl status kubelet
$ sudo systemctl restart kubelet
```
