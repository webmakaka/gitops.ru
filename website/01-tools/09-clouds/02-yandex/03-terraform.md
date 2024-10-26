---
layout: page
title: Yandex Clouds - Terraform
description: Yandex Clouds - Terraform
keywords: Deploy, Clouds, Yandex, Terraform
permalink: /tools/clouds/yandex/tools/terraform/
---

# Yandex Clouds - Terraform

Как использовать спецификации Terraform
Инфраструктура разворачивается в три этапа:

1. Команда terraform init инициализирует провайдеров, указанных в файле спецификации.

2. Команда terraform plan запускает проверку спецификации. Если есть ошибки — появятся предупреждения. Если ошибок нет, отобразится список элементов, которые будут созданы или удалены.

3. Команда terraform apply запускает развёртывание инфраструктуры.

Если инфраструктура не нужна, её можно уничтожить командой terraform destroy.

<br/>

# Практическая работа. Создаём виртуальную машину из образа и базу данных

https://learn.hashicorp.com/tutorials/tools/terraform/install-cli

<br/>

```
$ cd ~/tmp
$ vi my-config.tf
```

<br/>

```
$ yc config list
$ yc vpc subnet list
```

<br/>

```
variable "image-id" {
  type = string
}

resource "yandex_compute_instance" "vm-1" {
  name = "from-terraform-vm"
  platform_id = "standard-v1"
  zone = "ru-central1-a"

  resources {
    cores  = 2
    memory = 2
  }

  boot_disk {
    initialize_params {
      image_id = var.image-id
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.subnet-1.id
    nat       = true
  }

  metadata = {
    ssh-keys = "ubuntu:${file("~/.ssh/id_rsa.pub")}"
  }
}

resource "yandex_vpc_network" "network-1" {
  name = "from-terraform-network"
}

resource "yandex_vpc_subnet" "subnet-1" {
  name           = "from-terraform-subnet"
  zone           = "ru-central1-a"
  network_id     = yandex_vpc_network.network-1.id
  v4_cidr_blocks = ["10.2.0.0/16"]
}

output "internal_ip_address_vm_1" {
  value = yandex_compute_instance.vm-1.network_interface.0.ip_address
}

output "external_ip_address_vm_1" {
  value = yandex_compute_instance.vm-1.network_interface.0.nat_ip_address
}
```

```
$ vi my-variables.tfvars
```

<br/>

```
image-id = "идентификатор образа"
```

<br/>

```
$ terraform init
```

<br/>

```
$ terraform apply -var-file=my-variables.tfvars
```

<br/>

```
terraform state list
```

### Добавляем базу:

```
resource "yandex_mdb_postgresql_cluster" "postgres-1" {
  name        = "postgres-1"
  environment = "PRESTABLE"
  network_id  = yandex_vpc_network.network-1.id

  config {
    version = 12
    resources {
      resource_preset_id = "s2.micro"
      disk_type_id       = "network-ssd"
      disk_size          = 16
    }
    postgresql_config = {
      max_connections                   = 395
      enable_parallel_hash              = true
      vacuum_cleanup_index_scale_factor = 0.2
      autovacuum_vacuum_scale_factor    = 0.34
      default_transaction_isolation     = "TRANSACTION_ISOLATION_READ_COMMITTED"
      shared_preload_libraries          = "SHARED_PRELOAD_LIBRARIES_AUTO_EXPLAIN,SHARED_PRELOAD_LIBRARIES_PG_HINT_PLAN"
    }
  }

  database {
    name  = "postgres-1"
    owner = "my-name"
  }

  user {
    name       = "my-name"
    password   = "Test1234"
    conn_limit = 50
    permission {
      database_name = "postgres-1"
    }
    settings = {
      default_transaction_isolation = "read committed"
      log_min_duration_statement    = 5000
    }
  }

  host {
    zone      = "ru-central1-a"
    subnet_id = yandex_vpc_subnet.subnet-1.id
  }
}
```

<br/>

```
terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
}

provider "yandex" {
  token  =  "XXXXXXXXXXXXXXXXXXXXXXX"
  cloud_id  = "b1gttd235imdk2fdud9p"
  folder_id = "b1gfdbij3ijgopgqv9m9"
  zone      = "ru-central1-a"
}

variable "image-id" {
  type = string
}

resource "yandex_compute_instance" "vm-1" {
  name = "from-terraform-vm"
  platform_id = "standard-v1"
  zone = "ru-central1-a"

  resources {
    cores  = 2
    memory = 2
  }

  boot_disk {
    initialize_params {
      image_id = var.image-id
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.subnet-1.id
    nat       = true
  }

  metadata = {
    ssh-keys = "ubuntu:${file("~/.ssh/id_rsa.pub")}"
  }
}

resource "yandex_vpc_network" "network-1" {
  name = "from-terraform-network"
}

resource "yandex_vpc_subnet" "subnet-1" {
  name           = "from-terraform-subnet"
  zone           = "ru-central1-a"
  network_id     = yandex_vpc_network.network-1.id
  v4_cidr_blocks = ["10.2.0.0/16"]
}

resource "yandex_mdb_postgresql_cluster" "postgres-1" {
  name        = "postgres-1"
  environment = "PRESTABLE"
  network_id  = yandex_vpc_network.network-1.id

  config {
    version = 12
    resources {
      resource_preset_id = "s2.micro"
      disk_type_id       = "network-ssd"
      disk_size          = 16
    }
    postgresql_config = {
      max_connections                   = 395
      enable_parallel_hash              = true
      vacuum_cleanup_index_scale_factor = 0.2
      autovacuum_vacuum_scale_factor    = 0.34
      default_transaction_isolation     = "TRANSACTION_ISOLATION_READ_COMMITTED"
      shared_preload_libraries          = "SHARED_PRELOAD_LIBRARIES_AUTO_EXPLAIN,SHARED_PRELOAD_LIBRARIES_PG_HINT_PLAN"
    }
  }

  database {
    name  = "postgres-1"
    owner = "my-name"
  }

  user {
    name       = "my-name"
    password   = "Test1234"
    conn_limit = 50
    permission {
      database_name = "postgres-1"
    }
    settings = {
      default_transaction_isolation = "read committed"
      log_min_duration_statement    = 5000
    }
  }

  host {
    zone      = "ru-central1-a"
    subnet_id = yandex_vpc_subnet.subnet-1.id
  }
}

output "internal_ip_address_vm_1" {
  value = yandex_compute_instance.vm-1.network_interface.0.ip_address
}

output "external_ip_address_vm_1" {
  value = yandex_compute_instance.vm-1.network_interface.0.nat_ip_address
}
```

<br/>

Поскольку спецификация теперь включает создание БД, команда может выполняться довольно долго: около 10 минут.

В консоли откройте раздел Managed Service for PostgreSQL и убедитесь, что кластер postgres-1 создан и имеет статус Alive.

<br/>

```
$ terraform plan -var-file=my-variables.tfvars
$ terraform apply -var-file=my-variables.tfvars
$ terraform state list
$ terraform destroy -var-file=my-variables.tfvars
```
