---
layout: page
title: Беслпатное облако Google для запуска примеров с kubernetes в minikube
description: Беслпатное облако Google для запуска примеров с kubernetes в minikube
keywords: gitops, containers, kubernetes, setup, google cloud shell
permalink: /tools/containers/kubernetes/google-cloud-shell/
---

# Беслпатное облако Google для запуска примеров с kubernetes в minikube

<br/>

**Делаю:**  
2024.03.30

<br/>

Нужно иметь гуглопочту. Почти у всех есть.

Если 15-20 минут ничего не делать. Виртуалка удаляется.
Под домашний каталог дается что-то около 5GB. Эти данные остаются и не удаляются при удалении виртуалки.

4 ядра. 16 GB озу.

То, что перестартовывается, возможно, что даже и к лучшему.

<br/>

### Подключение к бесплатному облаку от Google

<br/>

#### Подключиться по https

https://shell.cloud.google.com/

<br/>

### Подключиться по ssh

<br/>

**Инструкция:**  
https://cloud.google.com/sdk/docs/install

<br/>

**Инсталлим google-cloud-sdk**

<br/>

```
$ cd ~/tmp

$ curl -O https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-470.0.0-linux-x86_64.tar.gz

$ tar -zxvf google-cloud-cli-470.0.0-linux-x86_64.tar.gz

$ cd google-cloud-sdk/

$ ./install.sh

$ source ~/.bashrc
```

<br/>

```
$ gcloud auth login
$ gcloud cloud-shell ssh
```

<br/>

### Получить доступ к сервису HTTP, запущенному в бесплатном облаке google

В веб консоли есть возможность открыть порт, но только для себя. Т.е. удаленные клиенты не смогут подключиться.

Вверху preview on port 8080

<br/>

### Получить доступ к сервису HTTP, запущенному в бесплатном облаке google с помощью ngrok

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

```
// Пример
$ kubectl --namespace logging port-forward deployment/kibana-kibana 8080:5601
```

<br/>

### Получить доступ к сервису HTTP, запущенному в бесплатном облаке google с помощью ngrok и с использованием Ingress

<br/>

Пример из ранее подготовленного примера.

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
$ export INGRESS_HOST=$(minikube --profile ${PROFILE} ip)
$ echo ${INGRESS_HOST}
```

<br/>

**kibana-ingress**

<br/>

```yaml
$ cat << EOF | kubectl apply -f -
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
    - host: ${INGRESS_HOST}.nip.io
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
// OK!
$ curl -I ${INGRESS_HOST}.nip.io
```

<br/>

```
// Удалить, если не нужен
// $ kubectl delete inggress kibana-ingress
```

<br/>

```
$ ./ngrok http ${INGRESS_HOST}.nip.io:80 --host-header=${INGRESS_HOST}.nip.io
```

<br/>

```
// OK!
http://60ff-34-147-0-94.ngrok.io
```
