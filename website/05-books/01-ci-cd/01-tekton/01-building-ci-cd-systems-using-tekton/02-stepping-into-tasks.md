---
layout: page
title: Building CI/CD Systems Using Tekton - Chapter 4. Stepping into Tasks
description: Building CI/CD Systems Using Tekton - Chapter 4. Stepping into Tasks
keywords: books, ci-cd, tekton, Chapter 4. Stepping into Tasks
permalink: /books/ci-cd/tekton/building-ci-cd-systems-using-tekton/stepping-into-tasks/
---

# Chapter 4. Stepping into Tasks

<br/>

### Building your first task

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

<br/>

### Adding additional Steps

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: multiple-steps
spec:
  steps:
    - name: first
      image: registry.access.redhat.com/ubi8/ubi-minimal
      command:
        - /bin/bash
        - -c
        - echo "First step running"
    - name: second
      image: alpine
      command:
        - /bin/sh
        - -c
        - echo "Second step running"
EOF
```

<br/>

```
$ tkn task start multiple-steps --showlog
TaskRun started: multiple-steps-run-nsxsl
Waiting for logs to be available...
[first] First step running

[second] Second step running

```

<br/>

### Using scripts

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: script
spec:
  steps:
    - name: step-with-script
      image: registry.access.redhat.com/ubi8/ubi
      script: |
        #!/usr/bin/env bash
        echo "Installing necessary tooling"
        dnf install iputils -y
        ping redhat.com -c 5
        echo "All done!"
EOF
```

<br/>

```
$ tkn task start script --showlog
```

<br/>

### Using scripts

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: script-node
spec:
  steps:
    - image: node:14
      script: |
        #!/usr/bin/env node
        console.log("This is some JS code");
EOF
```

<br/>

```
$ tkn task start script-node --showlog
```

<br/>

### Adding task parameters

(Не передался параметр)

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: hello-param
spec:
  params:
    - name: who
      type: string
  steps:
    - image: registry.access.redhat.com/ubi8/ubi-minimal
      command:
        - /bin/bash
        - -c
        - echo "Hello $(params.who)"
EOF
```

<br/>

```
$ tkn task start hello-param --showlog
```

<br/>

```
$ tkn task start hello-param --showlog -p who=Marley
```

<br/>

### Using array type parameters

(Не передались параметры)

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: groceries
spec:
  params:
    - name: grocery-items
      type: array
  steps:
    - name: grocery-list
      image: node:14
      args:
        - $(params.grocery-items[*])
      script: |
        #!/usr/bin/env node
        const items = process.argv.splice(2);
        console.log("Grocery List");
        items.map(i => console.log(`=> ${i}`));
EOF
```

<br/>

```
$ tkn task start groceries --showlog
```

<br/>

### Adding a default value

(Не отработало)

Надо как-то экранировать $(params.who)

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: hello-param
spec:
  params:
    - name: who
      type: string
      default: World
  steps:
    - image: registry.access.redhat.com/ubi8/ubi-minimal
      command:
        - /bin/bash
        - -c
        - echo "Hello $(params.who)"
EOF
```

<br/>

```
$ tkn task start hello-param --showlog --use-param-defaults
```

<br/>

### Sharing data

(Не отработало)

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: shared-home
spec:
  steps:
    - name: write
      image: registry.access.redhat.com/ubi8/ubi-minimal
      script: |
        cd ~
        echo Getting ready to write to $(pwd)
        echo "Secret Message" > message.txt
    - name: read
      image: registry.access.redhat.com/ubi8/ubi-minimal
      command:
        - /bin/bash
      args: ["-c", "cat ~/message.txt"]
EOF
```

<br/>

```
$ tkn task start shared-home --showlog
```

<br/>

### Using results

Results can be used in tasks to store the output from a task in a single file. Ultimately, the result is the same as using the home folder directly, but it is a standard way of sharing limited pieces of data across steps or even between tasks.

<br/>

Надо как-то экранировать $(results.message.path)

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: using-results
spec:
  results:
    - name: message
      description: Message to be shared
  steps:
    - name: write
      image: registry.access.redhat.com/ubi8/ubi-minimal
      command:
        - /bin/bash
      args:
        - "-c"
        - echo "Secret Message" | base64 > $(results.message.path)
    - name: read
      image: registry.access.redhat.com/ubi8/ubi-minimal
      command:
        - /bin/bash
      args:
        - "-c"
        - cat $(results.message.path)
EOF
```

<br/>

```
$ tkn task start using-results --showlog
```

<br/>

### Using Kubernetes volumes

1 не работает
2 непонятно, как должно работать

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: colors-map
data:
  error: "\e[31m"
  info: "\e[34m"
  debug: "\e[32m"
---
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: configmap
spec:
  volumes:
    - name: colors
      configMap:
        name: colors-map
  steps:
    - name: log-stuff
      image: registry.access.redhat.com/ubi8/ubi-minimal
      volumeMounts:
        - name: colors
          mountPath: /var/colors
      script: |
        echo $(cat /var/colors/info)Logging information
        echo $(cat /var/colors/debug)Debugging statement
        echo $(cat /var/colors/error)Colourized error statement
EOF
```

<br/>

```
$ tkn task start configmap --showlog
```

<br/>

### Digging into TaskRuns

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: failing
spec:
  steps:
    - image: nonexistinguser/notavalidcontainer
      command:
        - echo "Hello World"
EOF
```

<br/>

```
$ tkn task start failing
```

<br/>

```
$ kubectl get tr failing-run-mgk2t -o yaml
```