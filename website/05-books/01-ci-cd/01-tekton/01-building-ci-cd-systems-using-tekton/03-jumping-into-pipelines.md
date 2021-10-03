---
layout: page
title: Building CI/CD Systems Using Tekton - Jumping into Pipelines
description: Building CI/CD Systems Using Tekton - Jumping into Pipelines
keywords: books, ci-cd, tekton, Jumping into Pipelines
permalink: /books/ci-cd/tekton/building-ci-cd-systems-using-tekton/jumping-into-pipelines/
---

# Chapter 5. Jumping into Pipelines

<br/>

Пришлось пересоздать minikube.

<br/>

### Building your first pipeline

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: first-task
spec:
  steps:
    - image: registry.access.redhat.com/ubi8/ubi-minimal
      command:
        - /bin/bash
      args: ['-c', 'echo Hello from first task']
---
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: hello
spec:
  tasks:
    - name: first
      taskRef:
        name: first-task
EOF
```

<br/>

```
$ tkn pipeline start hello --showlog
```

<br/>

### Примерчик состоящий из 2-х тасок

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: die-roll
spec:
  steps:
    - name: greetings
      image: registry.access.redhat.com/ubi8/ubi-minimal
      command:
        - /bin/bash
      args: ['-c', 'echo Rolling 6-sided dice']
    - name: generate-random-number
      image: node:14
      script: |
        #!/usr/bin/env node
        const max = 6
        let randomNumber =  Math.floor(Math.random() * Math.floor(max));
        console.log(randomNumber + 1);
EOF
```

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: hello-dice-roll
spec:
  tasks:
    - name: first
      taskRef:
        name: first-task
    - name: roll
      taskRef:
        name: die-roll
EOF
```

<br/>

```
$ tkn pipeline start hello-dice-roll --showlog
```

<br/>

### Parameterizing pipelines

Ошибка! Не отработало.

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: die-roll-param
spec:
  params:
    - name: sides
      description: Number of sides to the dice
      default: "6"
      type: string
  steps:
    - name: greetings
      image: registry.access.redhat.com/ubi8/ubi-minimal
      command:
        - /bin/bash
      args: ['-c', 'echo Rolling $(params.sides)-sided dice']
    - name: generate-random-number
      image: node:14
      script: |
        #!/usr/bin/env node
        const max = $(params.sides)
        let randomNumber =  Math.floor(Math.random() * Math.floor(max));
        console.log(randomNumber + 1);
EOF
```

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: parametrized-dice-roll
spec:
  tasks:
    - name: first
      taskRef:
        name: first-task
    - name: roll
      taskRef:
        name: die-roll-param
EOF
```

<br/>

```
$ tkn pipeline start parametrized-dice-roll --showlog
```

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: parametrized-dice-roll
spec:
  tasks:
    - name: first
      taskRef:
        name: first-task
    - name: roll
      params:
        - name: sides
          value: "8"
      taskRef:
        name: die-roll-param
EOF
```

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: parametrized-dice-roll
spec:
  params:
    - name: dice-sides
      type: "string"
      default: "6"
      description: Number of sides on the dice
  tasks:
    - name: first
      taskRef:
        name: first-task
    - name: roll
      params:
        - name: sides
          value: "$(params.dice-sides)"
      taskRef:
        name: die-roll-param
EOF
```

<br/>

```
$ tkn pipeline start parametrized-dice-roll -p dice-sides=12 --showlog
```

<br/>

```
$ tkn pipeline start parametrized-dice-roll --use-param-defaults --showlog
```

<br/>

### Reusing tasks in the context of a pipeline

Ошибка! Не отработало.

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: logger
spec:
  params:
    - name: text
      type: string
  steps:
    - name: log
      image: registry.access.redhat.com/ubi8/ubi-minimal
      script: |
        DATE=$(date +%d/%m/%Y\ %T)
        echo [$DATE] - $(params.text)
EOF
```

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: task-reuse
spec:
  tasks:
    - name: say-hello
      params:
        - name: text
          value: "Hello"
      taskRef:
        name: logger
    - name: log-something
      params:
        - name: text
          value: "Something else being logged"
      taskRef:
        name: logger
EOF
```

<br/>

```
$ tkn pipeline start task-reuse --showlog
```

<br/>

### Ordering tasks within pipelines

Ошибка! Не отработало.

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: sleep-and-log
spec:
  params:
    - name: task-name
      type: string
    - name: time
      type: string
      default: "1"
  steps:
    - name: init
      image: registry.access.redhat.com/ubi8/ubi-minimal
      command:
        - /bin/bash
      args:
        - "-c"
        - "echo [$(date '+%d/%m/%Y %T')] - Task $(params.task-name) Started"
    - name: sleep
      image: registry.access.redhat.com/ubi8/ubi-minimal
      command:
        - /bin/bash
      args:
        - -c
        - sleep $(params.time)
    - name: log
      image: registry.access.redhat.com/ubi8/ubi-minimal
      command:
        - /bin/bash
      args:
        - "-c"
        - "echo [$(date '+%d/%m/%Y %T')] - Task $(params.task-name) Completed"
EOF
```

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: ordered-tasks
spec:
  tasks:
    - name: first
      params:
        - name: task-name
          value: A
        - name: time
          value: "2"
      taskRef:
        name: sleep-and-log
    - name: second
      params:
        - name: task-name
          value: B
      taskRef:
        name: sleep-and-log
      runAfter:
        - first
    - name: third
      params:
        - name: task-name
          value: C
        - name: time
          value: "3"
      taskRef:
        name: sleep-and-log
      runAfter:
        - first
    - name: fourth
      params:
        - name: task-name
          value: D
      taskRef:
        name: sleep-and-log
      runAfter:
        - second
        - third
EOF
```

<br/>

```
$ tkn pipeline start ordered-tasks --showlog
```

<br/>

### Using task results in pipelines

Ошибка! Не отработало.

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: dice-roll-result
spec:
  params:
    - name: sides
      description: Number of sides to the dice
      default: "6"
      type: string
  results:
    - name: dice-roll
      description: Random number generated by the dice roll
  steps:
    - name: generate-random-number
      image: node:14
      script: |
        #!/usr/bin/env node
        const fs = require("fs");
        const max = $(params.sides)
        let randomNumber =  Math.floor(Math.random() * Math.floor(max));
        fs.writeFile("$(results.dice-roll.path)", randomNumber.toString(), () => {
          console.log("Dice rolled");
        });
EOF
```

<br/>

```yaml
$ cat <<EOF | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: results
spec:
  params:
    - name: sides
      default: "6"
      type: "string"
  tasks:
    - name: intro
      params:
        - name: text
          value: "Preparing to roll the $(params.sides)-sided dice"
      taskRef:
        name: logger
    - name: roll
      params:
        - name: sides
          value: $(params.sides)
      taskRef:
        name: dice-roll-result
      runAfter:
        - intro
    - name: result
      params:
        - name: text
          value: "Result from dice roll was $(tasks.roll.results.dice-roll)"
      taskRef:
        name: logger
      runAfter:
        - roll
EOF
```

<br/>

```
$ tkn pipeline start results --showlog
```

<br/>

### Introducing pipeline runs

<br/>

```
$ kubectl get pipelineruns
```

<br/>

```
// Delete
$ kubectl delete pipelinerun results-run-s8w2j
```

<br/>

```
$ kubectl get pipelinerun results-run-sb6lk -o yaml
```
