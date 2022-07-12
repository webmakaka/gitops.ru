---
layout: page
title: Разворачиваем Gitlab с помощью ansible-galaxy и ansible-playbook
description: Разворачиваем Gitlab с помощью ansible-galaxy и ansible-playbook
keywords: Разворачиваем Gitlab с помощью ansible-galaxy и ansible-playbook
permalink: /tools/ansible/gitlab/
---

# Разворачиваем Gitlab с помощью Ansible

Делаю  
12.04.2019

<br/>

    $ mkdir ~/ansible-galaxy && cd ~/ansible-galaxy

<br/>

    $ vi Vagrantfile

<br/>

```

VAGRANTFILE_API_VERSION = "2"
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.ssh.insert_key = false
  config.ssh.forward_agent = true

  config.vm.provision :hosts do |provisioner|

      provisioner.add_host '192.168.56.101', ['controller']

  end

  config.vm.define "controller" do |controller|
    controller.vm.box = 'ubuntu/bionic64'
    controller.vm.hostname = 'controller'

    controller.vm.network :private_network, ip: "192.168.56.101"
    controller.vm.network :forwarded_port, guest: 22, host: 10122, id: "ssh"

    controller.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      v.customize ["modifyvm", :id, "--memory", 8192]
      v.customize ["modifyvm", :id, "--name", "controller"]
    end

    controller.vm.provision :shell, path: "install-ansible.sh"

  end

  config.vm.provision :shell, path: "update-ubuntu-packages.sh"

end

```

<br/>

    $ vi install-ansible.sh

<br/>

```
#!/usr/bin/env bash

apt-add-repository -y ppa:ansible/ansible
apt update
apt install -y ansible

```

<br/>

    $ vi update-ubuntu-packages.sh

<br/>

```
#!/usr/bin/env bash

apt update && apt upgrade -y

```

<br/>

    $ ssh-add ~/.vagrant.d/insecure_private_key
    $ vagrant box update
    $ vagrant up

<br/>

    $ vagrant ssh controller

<br/>

    $ sudo su  -
    # adduser --disabled-password --gecos "" ansible
    # usermod -aG sudo ansible
    # passwd ansible

    # vi /etc/ssh/sshd_config
    PasswordAuthentication yes

    # service sshd reload

<br/>

    # su - ansible

    $ ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa -q -N ""

    $ ssh-copy-id ansible@controller

    // Проверка подключения без ввода пароля
    $ ssh ansible@controller
    exit

    $ cd ~
    $ git clone --depth=1 https://github.com/webmakaka/Hands-On-DevOps-with-Ansible/

<br/>

    $ cd /home/tools/ansible/Hands-On-DevOps-with-Ansible/infrastructure/

<br/>

    $ vi inventory

<br/>

```
[gitlab-ce]
controller
```

<br/>

    $ vi ansible.cfg

<br/>

```
[defaults]
remote_user = ansible
host_key_checking = false
inventory = inventory

[privilege_escalation]
become = True
become_method = sudo
become_user = root
becore_ask_pass = False
```

<br/>

    $ vi playbook.yaml

```yaml
- hosts: gitlab-ce
  become: yes
  roles:
    - geerlingguy.gitlab
```

<br/>

    $ ansible-galaxy install geerlingguy.gitlab

<br/>

    $ ansible-playbook playbook.yaml -K

<br/>

Остается подключиться:  
http://192.168.56.101/
