---
layout: page
title: Using Infrastructure as Code to Build Reproducible Systems with Terraform on DigitalOcean
description: Using Infrastructure as Code to Build Reproducible Systems with Terraform on DigitalOcean
keywords: clouds, digital ocean, terraform, load balancer
permalink: /devops/clouds/do/terraform/
---

<br/>

# [Webinar] Using Infrastructure as Code to Build Reproducible Systems with Terraform on DigitalOcean

<br/>

### Datacenter Regions

DigitalOcean's datacenters are in the following locations:

* NYC1, NYC2, NYC3: New York City, United States
* AMS2, AMS3: Amsterdam, the Netherlands
* SFO1, SFO2: San Francisco, United States
* SGP1: Singapore
* LON1: London, United Kingdom
* FRA1: Frankfurt, Germany
* TOR1: Toronto, Canada
* BLR1: Bangalore, India

<br/>

https://www.digitalocean.com/docs/platform/availability-matrix/

<br/>

<div align="center">
    <iframe width="853" height="480" src="https://www.youtube.com/embed/U5suIJwobiQ" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

https://github.com/Zelgius/Infrastructure-As-Code-Intro

<br/>

### Terraform Digital Ocean


Делаю:  
10.02.2020

Наверное лучше для начала посмотреть вот этот материал:

<div align="center">
    <iframe width="853" height="480" src="https://www.youtube.com/embed/videoseries?list=PLtK75qxsQaMIHQOaDd0Zl_jOuu1m3vcWO" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>


<br/>

https://github.com/groovemonkey/digitalocean-terraform

<br/>


Manage -> API -> Generate New Token

Token name: terraform-digitalocean

<br/>

Account --> Security --> ADD SSH KEY

<br/>

    $ mkdir ~/do-tf-project && cd ~/do-tf-project
    $ code .

<br/>

**provider.tf**

```
provider "digitalocean" {
  token = var.do_token
}

```

**variables.tf**

```
variable "do_token" {
  type        = string
  description = "Your DigitalOcean API token"
  default     = "ENTER VALUE"
}

variable "ssh_fingerprint" {
  type        = string
  description = "Your SSH key fingerprint"
  default     = "ENTER VALUE"
}

variable "pub_key" {
  type        = string
  description = "The path to your public SSH key"
  default     = "keys/dokey.pub"
}

variable "pvt_key" {
  type        = string
  description = "The path to your private SSH key"
  default     = "keys/dokey"
}
```

<br/>

**web1.tf**

```
resource "digitalocean_droplet" "web1" {
  image = "ubuntu-16-04-x64"
  name = "web1"
  region = "NYC1"
  size = "512mb"
  private_networking = true
  ssh_keys = [
    "${var.ssh_fingerprint}"
  ]
  connection {
    host = self.ipv4_address
    user = "root"
    type = "ssh"
    private_key = file(var.pvt_key)
    timeout = "2m"
  }
    provisioner "remote-exec" {
        inline = [
            "sudo apt-get update",
            "sudo apt-get install -y nginx"
        ]
    }
}
```

<br/>

**web2.tf**

```
resource "digitalocean_droplet" "web2" {
  image = "ubuntu-16-04-x64"
  name = "web2"
  region = "NYC1"
  size = "512mb"
  private_networking = true
  ssh_keys = [
    "${var.ssh_fingerprint}"
  ]
  connection {
    host = self.ipv4_address
    user = "root"
    type = "ssh"
    private_key = file(var.pvt_key)
    timeout = "2m"
  }
    provisioner "remote-exec" {
        inline = [
            "sudo apt-get update",
            "sudo apt-get install -y nginx"
        ]
    }
}
```

<br/>

    $ terraform init
    $ terraform plan
    $ terraform apply

<br/>

    $ terraform destroy

<br/>

### Шаг 2. Добавляем Load-Balanced

<br/>

**web1.tf**

```
resource "digitalocean_droplet" "web1" {
  image = "ubuntu-16-04-x64"
  name = "web1"
  region = "NYC1"
  size = "512mb"
  private_networking = true
  user_data = file("config/webuserdata.sh")
  ssh_keys = [
    "${var.ssh_fingerprint}"
  ]
  connection {
    host = self.ipv4_address
    user = "root"
    type = "ssh"
    private_key = file(var.pvt_key)
    timeout = "2m"
  }
}
```

<br/>

**web2.tf**

```
resource "digitalocean_droplet" "web2" {
  image = "ubuntu-16-04-x64"
  name = "web2"
  region = "NYC1"
  size = "512mb"
  private_networking = true
  user_data = file("config/webuserdata.sh")
  ssh_keys = [
    "${var.ssh_fingerprint}"
  ]
  connection {
    host = self.ipv4_address
    user = "root"
    type = "ssh"
    private_key = file(var.pvt_key)
    timeout = "2m"
  }
}
```

<br/>

**config/webuserdata.sh**

```
#!/bin/bash

apt-get -y update
apt-get -y install nginx
export HOSTNAME=$(curl -s http://169.254.169.254/metadata/v1/hostname)
export PUBLIC_IPV4=$(curl -s http://169.254.169.254/metadata/v1/interfaces/public/0/ipv4/address)
echo Hello from Droplet $HOSTNAME, with IP Address: $PUBLIC_IPV4 > /var/www/html/index.nginx-debian.html
```


    $ terraform plan
    $ terraform apply

<br/>

**haproxy-web.tf**


```
resource "digitalocean_droplet" "haproxy-web" {
    image = "ubuntu-16-04-x64"
    name = "haproxy-web"
    region = "nyc1"
    size = "512mb"
    private_networking = true
    ssh_keys = [
      "${var.ssh_fingerprint}"
    ]
    connection {
        host = "digitalocean_droplet.haproxy-web"
        user = "root"
        type = "ssh"
        private_key = file(var.pvt_key)
        timeout = "2m"
    }
    provisioner "remote-exec" {
        inline = [
          "sleep 25",
          "sudo apt-get update",
          "sudo apt-get -y install haproxy"
        ]
    }
    provisioner "file" {
      content     = data.template_file.haproxyconf.rendered
      destination = "/etc/haproxy/haproxy.cfg"
    }
    provisioner "remote-exec" {
        inline = [
          "sudo service haproxy restart"
        ]
    }
}
```


<br/>

**config/haproxy.cfg.tpl**


```
global
  maxconn 2048
  log /dev/log    local0
  log /dev/log    local1 notice
  chroot /var/lib/haproxy
  stats socket /run/haproxy/admin.sock mode 660 level admin
  stats timeout 30s
  user haproxy
  group haproxy
  daemon

  # Default SSL material locations
  ca-base /etc/ssl/certs
  crt-base /etc/ssl/private

  # Default ciphers to use on SSL-enabled listening sockets.
  # For more information, see ciphers(1SSL).
  # Generated 2018-04-07 with https://mozilla.github.io/server-side-tls/ssl-config-generator/
  ssl-default-bind-ciphers ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256
  ssl-default-bind-options no-sslv3 no-tlsv10 no-tlsv11 no-tls-tickets
  ssl-default-server-ciphers ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256
  ssl-default-server-options no-sslv3 no-tlsv10 no-tlsv11 no-tls-tickets

defaults
    log global
    mode    http
    option  httplog
    option  dontlognull
    option  forwardfor
    option  http-server-close
    stats enable
    stats uri /stats
    stats realm Haproxy\ Statistics
    stats auth hapuser:password!1234
    timeout connect 5000
    timeout client  50000
    timeout server  50000
    errorfile 400 /etc/haproxy/errors/400.http
    errorfile 403 /etc/haproxy/errors/403.http
    errorfile 408 /etc/haproxy/errors/408.http
    errorfile 500 /etc/haproxy/errors/500.http
    errorfile 502 /etc/haproxy/errors/502.http
    errorfile 503 /etc/haproxy/errors/503.http
    errorfile 504 /etc/haproxy/errors/504.http

frontend www-http
  bind :80
  default_backend web-backend

backend web-backend
  server web1 ${web1_priv_ip}:80 check
  server web2 ${web2_priv_ip}:80 check
```

<br/>

**templates.tf**

```
data "template_file" "haproxyconf" {
  template = "${file("${path.module}/config/haproxy.cfg.tpl")}"

  vars = {
    web1_priv_ip = "${digitalocean_droplet.web1.ipv4_address_private}"
    web2_priv_ip = "${digitalocean_droplet.web2.ipv4_address_private}"
  }
}
```


<br/>

    $ terraform init
    $ terraform plan
    $ terraform apply

<br/>

Балансер заработал.

<br/>

    $ terraform destroy