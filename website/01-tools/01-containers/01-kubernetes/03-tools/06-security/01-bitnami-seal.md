---
layout: page
title: Установка kubeseal
description: Установка kubeseal
keywords: linux, kubernetes, Установка kubeseal
permalink: /tools/containers/kubernetes/tools/security/bitnami-seal/
---

# Установка kubeseal

<br/>

**Делаю:**  
XX.05.2023

<br/>

https://github.com/bitnami-labs/sealed-secrets/releases

<br/>

```
$ cd ~/tmp
$ wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.20.5/kubeseal-0.20.5-linux-amd64.tar.gz
$ tar -xvzf kubeseal-0.20.5-linux-amd64.tar.gz
$ sudo mv kubeseal /usr/local/bin/kubeseal
$ sudo chmod +x /usr/local/bin/kubeseal
```

<br/>

```
kubeseal --version
kubeseal version: 0.20.5
```

<br/>

### Приблизительно как работать

```
$ echo api_key_2a6f1d23eabc482f9032165de5a8c7 | base64
```

<br/>

```
$ vi secret.yaml
```

<br/>

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: appsecret
type: Opaque
data:
  apikey: YXBpX2tleV8yYTZmMWQyM2VhYmM0ODJmOTAzMjE2NWRlNWE4Yzc=
```

<br/>

```
// Get the public key using
$ kubeseal --fetch-cert > publickey.pem
```

<br/>

```
// Encrypt the contents of the secret
$ kubeseal --format=yaml --cert=publickey.pem < secret.yaml > sealedsecret.yaml
```

<br/>

```
$ rm secret.yaml
$ rm publickey.pem
```

<br/>

```
$ kubectl exec -it container-id -- sh
$ echo $APIKEY
api_key_2a6f1d23eabc482f9032165de5a8c7
```
