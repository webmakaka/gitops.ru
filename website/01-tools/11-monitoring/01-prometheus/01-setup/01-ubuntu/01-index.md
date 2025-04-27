---
layout: page
title: Инсталляция VirtualBox 7.X в командной строке в Ubuntu 22.04
description: Инсталляция VirtualBox 7.X в командной строке в Ubuntu 22.04
keywords: server, linux, virtual, virtualbox, setup, ubuntu,command line
permalink: /tools/monitoring/prometheus/setup/ubuntu/
---

# Инсталляция VirtualBox 7.X в командной строке в Ubuntu 22.04

Делаю:  
2025.04.04

<br/>

Взято за основу [Cbtnuggets] [Knox Hutchinson] DevOps Tools Engineer (Exam 701-100) Online Training [ENG, 2024]

<br/>

### Install and Configure Node Exporter

<br/>

```
$ sudo apt update -y && sudo apt upgrade -y
```

<br/>

```
$ sudo useradd -r -s /bin/false node_exporter
```

<br/>

```
https://github.com/prometheus/node_exporter/releases
```

<br/>

```
$ mkdir ~/tmp
$ cd ~/tmp/
```

<br/>

```
$ curl -LO https://github.com/prometheus/node_exporter/releases/download/v1.9.1/node_exporter-1.9.1.linux-amd64.tar.gz

$ tar -xvf node_exporter-1.9.1.linux-amd64.tar.gz
$ cd node_exporter-1.9.1.linux-amd64/

$ sudo cp node_exporter /usr/local/bin/
```

<br/>

```
$ sudo vi /etc/systemd/system/node_exporter.service
```

<br/>

```
[Unit]
Description=Prometheus Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
ExecStart=/usr/local/bin/node_exporter
Restart=always

[Install]
WantedBy=multi-user.target
```

<br/>

```
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now node_exporter.service
$ systemctl status node_exporter.service
```

<br/>

```
$ curl http://localhost:9100/metrics
```

<br/>

### Install and Configure Prometheus

<br/>

```
$ sudo useradd --no-create-home --shell /bin/false prometheus
```

<br/>

```
$ cd ~/tmp/
```

<br/>

```
// Пофиг что rc
$ curl -LO https://github.com/prometheus/prometheus/releases/download/v3.3.0-rc.0/prometheus-3.3.0-rc.0.linux-amd64.tar.gz

$ tar -xvf prometheus-3.3.0-rc.0.linux-amd64.tar.gz
```

<br/>

```
$ sudo mkdir /etc/prometheus
$ sudo mkdir /var/lib/prometheus

$ sudo chown prometheus:prometheus /etc/prometheus
$ sudo chown prometheus:prometheus /var/lib/prometheus
```

<br/>

```
$ sudo mv prometheus.yml /etc/prometheus/
$ sudo mv prometheus /usr/local/bin/
$ sudo mv promtool /usr/local/bin/
```

<br/>

```
$ sudo vi /etc/systemd/system/prometheus.service
```

<br/>

```
[Unit]
Description=Prometheus Monitoring
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file=/etc/prometheus/prometheus.yml \
    --storage.tsdb.path=/var/lib/prometheus
Restart=always

[Install]
WantedBy=multi-user.target
```

<br/>

```
$ sudo vi /etc/prometheus/prometheus.yml
```

<br/>

```
  - job_name: "node_exporter"
    static_configs:
      - targets: ["localhost:9090"]
```

<br/>

```
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now prometheus.service
$ systemctl status prometheus.service
```

<br/>

```
$ ls /var/lib/prometheus/
chunks_head  lock  queries.active  wal
```

<br/>

### Install and Configure Grafana

https://grafana.com/docs/grafana/latest/setup-grafana/installation/debian/

<br/>

```
$ sudo useradd -r -s /bin/false grafana
```

<br/>

```
$ wget https://dl.grafana.com/oss/release/grafana-11.6.0.linux-amd64.tar.gz
$ tar -zxvf grafana-11.6.0.linux-amd64.tar.gz
```

<br/>

```
$ mv grafana-v11.6.0/ grafana
$ sudo mv grafana /usr/local/
```

<br/>

```
$ sudo chown -R grafana:users /usr/local/grafana
```

<br/>

```
$ /usr/local/grafana/bin/grafana-server --homepath /usr/local/grafana
$ sudo chown -R grafana:users /usr/local/grafana
```

<br/>

```
$ sudo vi /etc/systemd/system/grafana-server.service
```

<br/>

```
[Unit]
Description=Grafana Server
After=network.target

[Service]
Type=simple
User=grafana
Group=users
ExecStart=/usr/local/grafana/bin/grafana server --config=/usr/local/grafana/conf/grafana.ini --homepath=/usr/local/grafana
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

<br/>

```
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now grafana-server.service
$ systemctl status grafana-server.service
```

<br/>

```
$ sudo systemctl stop grafana-server.service
$ sudo systemctl start grafana-server.service
$ systemctl status grafana-server.service
```

<br/>

```
// admin / admin
http://localhost:3000
```

<br/>

```
// datasources
http://localhost:3000/connections/datasources/new

prometheus

connection: http://localhost:9090
```

<br/>

```
// dashboard
http://localhost:3000/dashboard/import

1860 - Load

Указать prometheus datasource

Import
```

<br/>

### Отключить все

<br/>

```
$ sudo systemctl stop node_exporter.service
$ sudo systemctl stop prometheus.service
$ sudo systemctl stop grafana-server.service
```
