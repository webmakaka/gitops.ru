---
layout: page
title: Building CI/CD Systems Using Tekton - Chapter 4. Stepping into Tasks
description: Building CI/CD Systems Using Tekton - Chapter 4. Stepping into Tasks
keywords: books, ci-cd, tekton, Chapter 4. Stepping into Tasks
permalink: /books/ci-cd/tekton/building-ci-cd-systems-using-tekton/stepping-into-tasks/
---

# Chapter 4. Stepping into Tasks

<br/>

https://github.com/PacktPublishing/Building-CI-CD-systems-using-Tekton/blob/main/chapter-4/hello.yaml

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: hello
spec:
  steps:
    - image: registry.access.redhat.com/ubi8/ubi-minimal
      command:
        - /bin/bash
        - -c
        - echo "Hello World"
EOF
```

<br/>

```
$  tkn task ls
NAME    DESCRIPTION   AGE
hello                 5 seconds ago

```

<br/>

```
$ tkn task start hello --showlog
TaskRun started: hello-run-6gwtx
Waiting for logs to be available...
[unnamed-0] Hello World
```
