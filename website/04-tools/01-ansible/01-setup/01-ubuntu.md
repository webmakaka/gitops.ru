---
layout: page
title: Инсталляция Ansible в Ubuntu 20.04
description: Инсталляция Ansible в Ubuntu 20.04
keywords: tools, ansible, setup, ubuntu
permalink: /tools/ansible/setup/ubuntu/
---

# Инсталляция Ansible в Ubuntu 20.04

Делаю:  
23.09.2021

    $ sudo apt-add-repository -y ppa:ansible/ansible

    $ sudo apt update && sudo apt install -y ansible

    $ ansible --version
    ansible 2.9.6
