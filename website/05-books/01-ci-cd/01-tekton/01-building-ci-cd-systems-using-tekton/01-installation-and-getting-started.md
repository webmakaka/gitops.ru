---
layout: page
title: Building CI/CD Systems Using Tekton - Chapter 3. Installation and Getting Started
description: Building CI/CD Systems Using Tekton - Chapter 3. Installation and Getting Started
keywords: books, ci-cd, tekton, Installation and Getting Started
permalink: /books/ci-cd/tekton/building-ci-cd-systems-using-tekton/installation-and-getting-started/
---

# Chapter 3. Installation and Getting Started

<br/>

1. Инсталляция [MiniKube](/containers/k8s/setup/minikube/) (Ingress и остальное можно не устанавливать)
2. Инсталляция [Kubectl](/containers/k8s/setup/tools/kubectl/)

3. ??? Инсталляция Git
4. ??? Инсталляция Docker
5. ??? Инсталляция Node.js

6. ??? Инсталляция VSCode

**Extensions:**

• Kubernetes ( ms-kubernetes-tools.vscode-kubernetes-tools ) by
Microsoft
• YAML ( redhat.vscode-yaml ) by Red Hat
• Tekton Pipelines ( redhat.vscode-tekton-pipelines ) by Red Hat

<br/>

#### Инсталляция Tekton CLI

<br/>

```

$ cd ~/tmp/

$ export LATEST_VERSION=$(curl --silent "https://api.github.com/repos/tektoncd/cli/releases/latest" | grep '"tag_name"' | sed -E 's/.*"([^"]+)".*/\1/')
$ export LATEST_VERSION_SHORT=$(curl --silent "https://api.github.com/repos/tektoncd/cli/releases/latest" | grep '"tag_name"' | sed -E 's/.*"([^"]+)".*/\1/' | cut -c 2-)

$ curl -LO "https://github.com/tektoncd/cli/releases/download/${LATEST_VERSION}/tkn_${LATEST_VERSION_SHORT}_$(uname -s)_$(uname -m).tar.gz"

$ sudo tar xvzf tkn_${LATEST_VERSION_SHORT}_$(uname -s)_$(uname -m).tar.gz -C /usr/local/bin/ tkn
```

<br/>

```
$ tkn version
Client version: 0.20.0
```

<br/>

```
$ kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
```

<br/>

#### Tekton Dashboard

<br/>

```
$ kubectl apply --filename https://github.com/tektoncd/dashboard/releases/latest/download/tekton-dashboard-release.yaml

$ kubectl --namespace tekton-pipelines port-forward svc/tekton-dashboard 8080:9097

```

https://shell.cloud.google.com/

Вверху справа 3-й значок слева

Preview on port 8080

Открывается окно Tekton Dashboard
