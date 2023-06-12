---
layout: page
title: GitOps Cookbook - Cloud Native CI/CD - Tekton
description: GitOps Cookbook - Cloud Native CI/CD - Tekton
keywords: GitOps Cookbook - Cloud Native CI/CD, Tekton
permalink: /books/gitops/gitops-cookbook/cloud-native-cicd/tekton/create-a-task-to-compile-and-package-an-app-from-private-git/
---

<br/>

# [Book] GitOps Cookbook: 06. Cloud Native CI/CD: Tekton

<br/>

## [OK!] 6.4 Create a Task to Compile and Package an App from Private Git

<br/>

Создаю private repo wildmakaka/tekton-greeter-private, скопировав содержимое https://github.com/gitops-cookbook/tekton-tutorial-greeter в ветку main.

<br/>

Settings -> Developer Settings -> Personal access token -> Tokens (classic) -> Genereate new token classic -> TEKTON_TOKEN

<br/>

```yaml
$ cat << 'EOF' | kubectl create -f -
apiVersion: v1
kind: Secret
metadata:
  name: github-secret
  annotations:
    tekton.dev/git-0: https://github.com
type: kubernetes.io/basic-auth
stringData:
  username: YOUR_USERNAME
  password: YOUR_PASSWORD
EOF
```

<br/>

You Git password, in this case your GitHub personal access token

<br/>

```yaml
$ cat << 'EOF' | kubectl create -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tekton-bot-sa
secrets:
  - name: github-secret
EOF
```

<br/>

```
// master на main нужно не забыть поменять
$ tkn task start build-app \
--serviceaccount='tekton-bot-sa' \
--param url='https://github.com/wildmakaka/wildmakaka-tekton-greeter-private.git' \
--param contextDir='quarkus' \
--workspace name=source,emptyDir="" \
--showlog
```

<br/>

```
[build-sources] [INFO] ------------------------------------------------------------------------
[build-sources] [INFO] BUILD SUCCESS
[build-sources] [INFO] ------------------------------------------------------------------------
[build-sources] [INFO] Total time:  37.627 s
[build-sources] [INFO] Finished at: 2023-05-29T12:25:16Z
[build-sources] [INFO] ------------------------------------------------------------------------
```
