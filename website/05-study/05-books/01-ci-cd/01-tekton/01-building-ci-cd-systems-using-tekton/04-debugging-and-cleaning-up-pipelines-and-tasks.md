---
layout: page
title: Building CI/CD Systems Using Tekton - Debugging and Cleaning Up Pipelines and Tasks
description: Building CI/CD Systems Using Tekton - Debugging and Cleaning Up Pipelines and Tasks
keywords: books, ci-cd, tekton, Debugging and Cleaning Up Pipelines and Tasks
permalink: /study/books/ci-cd/tekton/building-ci-cd-systems-using-tekton/debugging-and-cleaning-up-pipelines-and-tasks/
---

# Chapter 6. Debugging and Cleaning Up Pipelines and Tasks

<br/>

### Debugging pipelines

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: invalid-image
spec:
  steps:
    - name: log
      image: invaliduser/nonexistingimage
      command:
        - /bin/bash
      args: ['-c', 'echo "this task will fail"']
EOF
```

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: failing
spec:
  tasks:
    - name: fail
      taskRef:
        name: invalid-image
EOF
```

<br/>

```
$ tkn pipeline start failing --showlog
```

<br/>

```
$ tkn pipelinerun ls
```

<br/>

```
$ tkn pipelinerun logs failing-run-jldmw
```

<br/>

```
$ kubectl get pods
```

<br/>

```
$ kubectl describe pod/failing-run-jldmw-fail-58ng8-pod-9rtx5
```

<br/>

```
$ tkn pipelinerun cancel failing-run-jldmw
```

<br/>

### Установить TimeOut

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: failing
spec:
  tasks:
    - name: fail
      timeout: "0h0m30s"
      taskRef:
        name: invalid-image
EOF
```

<br/>

### Running a halting task

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: log-and-exit
spec:
  params:
    - name: text
      type: string
    - name: exitcode
      type: string
  steps:
    - name: log
      image: registry.access.redhat.com/ubi8/ubi
      command:
        - /bin/bash
      args: ['-c', 'echo $(params.text)']
    - name: exit
      image: registry.access.redhat.com/ubi8/ubi
      command:
        - /bin/bash
      args: [
        "-c",
        "echo 'Exiting with code $(params.exitcode)' && exit $(params.exitcode)"
      ]
EOF
```

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: exitcodes
spec:
  tasks:
    - name: clone
      taskRef:
        name: log-and-exit
      params:
        - name: text
          value: "Simulating git clone"
        - name: exitcode
          value: "0"
    - name: unit-tests
      taskRef:
        name: log-and-exit
      params:
        - name: text
          value: "Simulating unit testing"
        - name: exitcode
          value: "1"
      runAfter:
        - clone
    - name: deploy
      taskRef:
        name: log-and-exit
      params:
        - name: text
          value: "Simulating deployment"
        - name: exitcode
          value: "0"
      runAfter:
        - unit-tests
EOF
```

<br/>

```
$ tkn pipeline start exitcodes --showlog
```

<br/>

Нужно руками прописывать значение отличное от 0, чтобы поймать ошибку.

<br/>

```
$ tkn pipelinerun describe exitcodes-run-pdv5x
```

<br/>

```
$ kubectl -n default logs exitcodes-run-pdv5x-unit-tests-h722m-
pod-n5n77 -c step-exit
```

<br/>

### Adding a finally task

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: cleanup
spec:
  steps:
    - name: clean
      image: registry.access.redhat.com/ubi8/ubi
      command:
        - /bin/bash
      args: ['-c', 'echo Cleaning up!']
EOF
```

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: exitcodes
spec:
  tasks:
    - name: clone
      taskRef:
        name: log-and-exit
      params:
        - name: text
          value: "Simulating git clone"
        - name: exitcode
          value: "0"
    - name: unit-tests
      taskRef:
        name: log-and-exit
      params:
        - name: text
          value: "Simulating unit testing"
        - name: exitcode
          value: "1"
      runAfter:
        - clone
    - name: deploy
      taskRef:
        name: log-and-exit
      params:
        - name: text
          value: "Simulating deployment"
        - name: exitcode
          value: "0"
      runAfter:
        - unit-tests
  finally:
    - name: cleanup-task
      taskRef:
        name: cleanup
EOF
```

<br/>

```
$ tkn pipeline start exitcodes --showlog
```
