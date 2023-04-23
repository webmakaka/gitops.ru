---
layout: page
title: Terraform Setup
description: Terraform Setup
keywords: devops, tools, terraform, setup
permalink: /tools/terraform/setup/
---

# Terraform Setup

Делаю:  
19.08.2021

<br/>

```
// Лучше с сайта:
https://developer.hashicorp.com/terraform/downloads?product_intent=terraform
```

<br/>

### Установка terraform из github

**v1.0.5**

(Последняя на сегодня 1.4.5)

<br/>

```
$ echo LATEST_VERSION=$(curl --silent "https://api.github.com/repos/hashicorp/terraform/releases/latest" | grep '"tag_name"' | sed -E 's/.*"([^"]+)".*/\1/')

$ wget https://releases.hashicorp.com/tools/terraform/1.0.5/terraform_1.0.5_linux_amd64.zip

$ unzip terraform_1.0.5_linux_amd64.zip

$ sudo mv terraform /usr/local/bin/

$ terraform version
Terraform v1.0.5
```

<br/>

### Install and configure tfswitch

The tfswitch command line tool lets you switch between different versions of terraform

<br/>

```
$ wget https://github.com/warrensbox/terraform-switcher/releases/download/0.7.737/terraform-switcher_0.7.737_linux_amd64.tar.gz

$ mkdir -p ${HOME}/bin

$ tar -xvf terraform-switcher_0.7.737_linux_amd64.tar.gz -C ${HOME}/bin

$ export PATH=$PATH:${HOME}/bin

$ tfswitch -b ${HOME}/bin/terraform 0.11.14

$ echo "0.11.14" >> .tfswitchrc

$ exit

```
