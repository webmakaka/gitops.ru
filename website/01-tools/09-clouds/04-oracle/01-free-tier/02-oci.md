---
layout: page
title: Инсталляция OCI в ubuntu
description: Инсталляция OCI в ubuntu
keywords: devops, clouds, Oracle Clouds, oci
permalink: /tools/clouds/oracle/free-tier/oci/
---

# Инсталляция OCI в ubuntu

```
$ bash -c "$(curl -L https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.sh)"
```

<br/>

```
// Чтобы shell мог выполнять команды oci
$ exec -l $SHELL
```

<br/>

```
oci --version
3.4.2
```

<br/>

**Configure the OCI CLI**
https://www.youtube.com/watch?v=x2iWGXIa-rQ

<br/>

```
$ oci setup config
```

<br/>

```
$ ls -la ~/.oci/
```

<br/>

Oracle Cloud Web UI -> Profile -> User-Settigns -> API-Keys -> Add -> Choose Public Key File -> Add

<br/>

```
$ oci iam user list
```
