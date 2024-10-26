---
layout: page
title: Yandex Clouds - Packer
description: Yandex Clouds - Packer
keywords: Deploy, Clouds, Yandex, Packer
permalink: /tools/clouds/yandex/packer/
---

# Yandex Clouds - Packer

**Install Packer:**  
https://learn.hashicorp.com/tutorials/packer/get-started-install-cli

<br/>

https://practicum.yandex.ru/trainer/ycloud/lesson/1fa859a7-dae3-405f-b7b4-57e4e3f0fbf7/

<br/>

```
$ cd ~/tmp
$ vi my-ubuntu-nginx.pkr.hcl
```

<br/>

```
$ yc config list
$ yc vpc subnet list
```

<br/>

```
source "yandex" "ubuntu-nginx" {
  token               = "ваш OAuth-токен"
  folder_id           = "идентификатор каталога"
  source_image_family = "ubuntu-2004-lts"
  ssh_username        = "ubuntu"
  use_ipv4_nat        = "true"
  image_description   = "my custom ubuntu with nginx"
  image_family        = "ubuntu-2004-lts"
  image_name          = "my-ubuntu-nginx"
  subnet_id           = "идентификатор подсети"
  disk_type           = "network-ssd"
  zone                = "ru-central1-a"
}

build {
  sources = ["source.yandex.ubuntu-nginx"]

  provisioner "shell" {
    inline = ["sudo apt-get update -y",
              "sudo apt-get install -y nginx",
              "sudo systemctl enable nginx.service"]
  }
}
```

<br/>

```
$ packer build my-ubuntu-nginx.pkr.hcl
```

<br/>

Compute Cloud. Ищите образ в разделе Образы.
