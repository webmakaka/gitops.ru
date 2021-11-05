---
layout: page
title: Logging in Kubernetes with EFK Stack | The Complete Guide
description: Logging in Kubernetes with EFK Stack | The Complete Guide
keywords: containers, kubernetes, linode, elastic, fluentd, kibana
permalink: /study/videos/containers/kubernetes/tools/logging/efk/logging-in-kubernetes-with-efk-stack/
---

# [Nana Janashia] Logging in Kubernetes with EFK Stack | The Complete Guide [ENG, 2021]

<br/>

Делаю:  
04.11.2021

<br/>

### Ссылки

**node app:**  
https://gitlab.com/nanuchi/node-app

<br/>

**java app**
https://gitlab.com/nanuchi/java-app

<br/>

**Промо на облачный kubernetes от linode на $100**  
https://gitlab.com/nanuchi/efk-course-commands/-/tree/master

<br/>

**Еще какие-то полезные ссылки**  
https://gitlab.com/nanuchi/efk-course-commands/-/blob/master/links.md

<br/>

**Set up elastic stack in kubernetes cluster**  
https://gitlab.com/nanuchi/efk-course-commands/-/blob/master/commands.md

<br/>

### Подключение к бесплатному облаку от Google

https://shell.cloud.google.com/

<br/>

**Инсталлим google-cloud-sdk**

https://cloud.google.com/sdk/docs/install

<br/>

```
$ gcloud auth login
$ gcloud cloud-shell ssh
```

<br/>

1. Инсталляция [MiniKube](/containers/kubernetes/setup/minikube/)

**Испольновалась версия KUBERNETES_VERSION=v1.22.2**

2. Инсталляция [Kubectl](/containers/kubernetes/tools/kubectl/)

3. Инсталляция [helm](/containers/kubernetes/tools/helm/setup/)

4. Инсталляция [Elastic Search, Kibana, Fluentd](/containers/kubernetes/tools/helm/)

<br/>

### Подготавливаем образы и выкладываем на hub.docker.com

<br/>

```
$ export DOCKER_HUB_LOGIN=webmakaka
```

<br/>

Для меня образы в публичном регистри - норм.

<br/>

```
$ docker login
```

<br/>

**Приватные репо не нужны:**

<br/>

**node-app**

```
$ cd ~/tmp
$ git clone https://gitlab.com/nanuchi/node-app.git
$ cd node-app
$ docker build -t node-app .
$ docker tag node-app ${DOCKER_HUB_LOGIN}/node-1.0:latest
$ docker push ${DOCKER_HUB_LOGIN}/node-1.0
```

<br/>

**java-app**

```
$ cd ~/tmp
$ git clone https://gitlab.com/nanuchi/java-app.git
$ cd java-app
$ ./gradlew build
$ docker build -t java-app .
$ docker tag java-app ${DOCKER_HUB_LOGIN}/java-1.0:latest
$ docker push ${DOCKER_HUB_LOGIN}/java-1.0
```

<br/>

### Create docker-registry secret for dockerHub (Пропускаю)

Не нужно выполнять, если image хранятся в публичном registry.

<br/>

```
$ export DOCKER_REGISTRY_SERVER=docker.io
$ export DOCKER_USER=your dockerID, same as for `docker login`
$ export DOCKER_EMAIL=your dockerhub email, same as for `docker login`
$ export DOCKER_PASSWORD=your dockerhub pwd, same as for `docker login`

$ kubectl create secret docker-registry myregistrysecret \
--docker-server=${DOCKER_REGISTRY_SERVER} \
--docker-username=${DOCKER_USER} \
--docker-password=${DOCKER_PASSWORD} \
--docker-email=${DOCKER_EMAIL}

$ kubectl get secret
```

<br/>

### Deploy

Оригинальные лежат в репо проектов.

<br/>

**node-app**

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-app
  labels:
    app: node-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: node-app
  template:
    metadata:
      labels:
        app: node-app
    spec:
      containers:
        - name: node-app
          image: ${DOCKER_HUB_LOGIN}/node-1.0
          imagePullPolicy: Always
          ports:
            - containerPort: 3000
EOF
```

<br/>

**java-app**

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-app
  labels:
    app: java-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: java-app
  template:
    metadata:
      labels:
        app: java-app
    spec:
      containers:
        - name: java-app
          image: ${DOCKER_HUB_LOGIN}/java-1.0
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
EOF
```

<br/>

```
$ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
java-app-85b44765bb-rqlwk   1/1     Running   0          6s
node-app-6c87fddb75-wn285   1/1     Running   0          12s
```

<br/>

```
$ kubectl --namespace logging get pods
NAME                             READY   STATUS    RESTARTS      AGE
elasticsearch-master-0           1/1     Running   0             3m58s
elasticsearch-master-1           1/1     Running   0             3m58s
elasticsearch-master-2           1/1     Running   0             3m58s
fluentd-0                        1/1     Running   0             2m44s
fluentd-hvntt                    1/1     Running   3 (48s ago)   2m45s
kibana-kibana-56689685dc-8prxl   1/1     Running   0             3m17s
```

<br/>

### Проверка работы без использования ingress

```
$ kubectl --namespace logging port-forward deployment/kibana-kibana 8080:5601
```

<br/>

**Подключаемся еще 1 терминалом**

```
$ gcloud cloud-shell ssh
```

<br/>

Нужно зарегаться  
https://ngrok.com/download

<br/>

```
$ cd ~/tmp
$ wget https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-amd64.zip
$ unzip ngrok-stable-linux-amd64.zip
$ ./ngrok authtoken <YOUR_TOKEN>
```

<br/>

```
$ cd ~/tmp
$ ./ngrok http 8080
```

<br/>

**Подключаемся еще 1 терминалом**

```
$ gcloud cloud-shell ssh
```

<br/>

```
$ curl -I localhost:8080
OK!
```

<br/>

**Подключаюсь извне:**

<br/>

```
http://ba8e-34-91-125-232.ngrok.io
OK!
```

<br/>

Убираю port-forward

<br/>

### Публикация сервиса в интернет с помощью Ingress (хз зачем, ведь и так работает)

<br/>

```
$ kubectl --namespace logging get svc
NAME                            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)              AGE
elasticsearch-master            ClusterIP   10.106.92.197   <none>        9200/TCP,9300/TCP    4m55s
elasticsearch-master-headless   ClusterIP   None            <none>        9200/TCP,9300/TCP    4m55s
fluentd-aggregator              ClusterIP   10.108.30.236   <none>        9880/TCP,24224/TCP   3m42s
fluentd-forwarder               ClusterIP   10.97.175.200   <none>        9880/TCP             3m42s
fluentd-headless                ClusterIP   None            <none>        9880/TCP,24224/TCP   3m42s
kibana-kibana                   ClusterIP   10.105.41.51    <none>        5601/TCP             4m14s
```

<br/>

**Публикация сервиса kibana-kibana в неймспейсе logging, чтобы к нему можно было обращаться из ingress**

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
kind: Service
apiVersion: v1
metadata:
  name: kibana-service
spec:
  type: ExternalName
  externalName: kibana-kibana.logging.svc.cluster.local
EOF
```

<br/>

```
$ minikube --profile marley-minikube ip
192.168.49.2
```

<br/>

**kibana-ingress**

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/default-backend: ingress-nginx-controller
    kubernetes.io/ingress.class: nginx
    ## tells ingress to check for regex in the config file
    nginx.ingress.kubernetes.io/use-regex: "true"
  name: kibana-ingress
spec:
  rules:
    - host: 192.168.49.2.nip.io
      http:
        paths:
          - pathType: Prefix
            path: /
            backend:
              service:
                name: kibana-service
                port:
                  number: 5601
EOF
```

<br/>

```
$ kubectl get ingress
NAME             CLASS    HOSTS                 ADDRESS   PORTS   AGE
kibana-ingress   <none>   192.168.49.2.nip.io             80      5s
```

<br/>

```
$ curl -I 192.168.49.2.nip.io
OK!
```

<br/>

```
// Удалить, если не нужен
// $ kubectl delete inggress kibana-ingress
```

<br/>

```
./ngrok http 192.168.49.2.nip.io:80 --host-header=192.168.49.2.nip.io
```

<br/>

http://60ff-34-147-0-94.ngrok.io

OK!

<br/>

### Настройка Fluentd

config maps -> fluentd-forwarder

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
    name: fluentd-forwarder-cm
    namespace: default
    labels:
        app.kubernetes.io/component: forwarder
        app.kubernetes.io/instance: fluentd
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/name: fluentd
        helm.sh/chart: fluentd-1.3.0
    annotations:
        meta.helm.sh/release-name: fluentd
        meta.helm.sh/release-namespace: default
data:
    fluentd.conf: |

        # Ignore fluentd own events
        <match fluent.**>
            @type null
        </match>

        # HTTP input for the liveness and readiness probes
        <source>
            @type http
            port 9880
        </source>

        # Throw the healthcheck to the standard output instead of forwarding it
        <match fluentd.healthcheck>
            @type null
        </match>

        # Get the logs from the containers running in the node
        <source>
          @type tail
          path /var/log/containers/*-app*.log
          pos_file /opt/bitnami/fluentd/logs/buffers/fluentd-docker.pos
          tag kubernetes.*
          read_from_head true
          format json
          time_format %Y-%m-%dT%H:%M:%S.%NZ
        </source>

        <filter **>
          @type parser
          key_name log
          <parse>
            @type multi_format
            <pattern>
              format json
              time_key time
              keep_time_key true
            </pattern>
          </parse>
        </filter>

        # enrich with kubernetes metadata
        <filter kubernetes.**>
            @type kubernetes_metadata
        </filter>


        <match kubernetes.var.log.containers.**java-app**.log>
          @type elasticsearch
          include_tag_key true
          host "elasticsearch-master.default.svc.cluster.local"
          port "9200"
          index_name "java-app-logs"
          <buffer>
            @type file
            path /opt/bitnami/fluentd/logs/buffers/java-logs.buffer
            flush_thread_count 2
            flush_interval 5s
          </buffer>
        </match>

        <match kubernetes.var.log.containers.**node-app**.log>
          @type elasticsearch
          include_tag_key true
          host "elasticsearch-master.default.svc.cluster.local"
          port "9200"
          index_name "node-app-logs"
          <buffer>
            @type file
            path /opt/bitnami/fluentd/logs/buffers/node-logs.buffer
            flush_thread_count 2
            flush_interval 5s
          </buffer>
        </match>
```

<br/>

```
$ kubectl rollout restart daemonset/fluentd
```

<br/>

Kibana -> Index Patterns -> Create Index pattern

Index pattern name -> _app_ -> Next step

Fime field -> time

Create index pattern

<br/>

Kibana -> Discovery

<br/>
