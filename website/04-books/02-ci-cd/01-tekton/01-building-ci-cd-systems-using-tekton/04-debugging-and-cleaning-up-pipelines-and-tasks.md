---
layout: page
title: Building CI/CD Systems Using Tekton - Debugging and Cleaning Up Pipelines and Tasks
description: Building CI/CD Systems Using Tekton - Debugging and Cleaning Up Pipelines and Tasks
keywords: books, ci-cd, tekton, Debugging and Cleaning Up Pipelines and Tasks
permalink: /books/ci-cd/tekton/building-ci-cd-systems-using-tekton/debugging-and-cleaning-up-pipelines-and-tasks/
---

# Chapter 6. Debugging and Cleaning Up Pipelines and Tasks

<br/>

### Debugging pipelines

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
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
$ cat << 'EOF' | kubectl apply -f -
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
$ cat << 'EOF' | kubectl apply -f -
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
$ cat << 'EOF' | kubectl apply -f -
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
$ cat << 'EOF' | kubectl apply -f -
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
$ cat << 'EOF' | kubectl apply -f -
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
$ cat << 'EOF' | kubectl apply -f -
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

<br/>

## Assessments

<br/>

### Fail if root

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: fail-if-root
spec:
  steps:
    - name: fail-if-root
      image: registry.access.redhat.com/ubi8/ubi
      script: |
        if [ $(whoami) == "root" ]
          then
            echo "User is root"
            exit 1
        fi
        exit 0
---
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: check-root
spec:
  tasks:
    - name: is-root
      taskRef:
        name: fail-if-root
EOF
```

<br/>

```
$ tkn pipeline start check-root --showlog
```

<br/>

```
PipelineRun started: check-root-run-s54n4
Waiting for logs to be available...
[is-root : fail-if-root] ++ whoami
[is-root : fail-if-root] User is root
[is-root : fail-if-root] + '[' root == root ']'
[is-root : fail-if-root] + echo 'User is root'
[is-root : fail-if-root] + exit 1

failed to get logs for task is-root : container step-fail-if-root has failed  : [{"key":"StartedAt","value":"2021-10-17T14:41:34.467Z","type":3}]
```

<br/>

### Make your bets

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: draw-card
spec:
  results:
    - name: card
      description: Card value
  steps:
    - name: randomize
      image: node:14
      script: |
        #!/usr/bin/env node
        const fs = require("fs");
        const cardValue = Math.floor(Math.random() * 10) + 1;
        fs.writeFileSync("$(results.card.path)", cardValue.toString());
---
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: check-result
spec:
  params:
    - name: new-card
      type: string
  steps:
    - name: add-card
      image: registry.access.redhat.com/ubi8/ubi
      script: |
        HAND=17
        NEWCARD=$(params.new-card)
        NEWHAND=$(($HAND+$NEWCARD))
        echo "New hand value is $NEWHAND"
        if (($NEWHAND > 21))
          then
            echo "Busted"
            exit 1
        fi
        echo "You won"
        exit 0
---
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: make-your-bets
spec:
  tasks:
    - name: draw
      taskRef:
        name: draw-card
    - name: check
      taskRef:
        name: check-result
      params:
        - name: new-card
          value: $(tasks.draw.results.card)
      runAfter:
        - draw
  finally:
    - name: clean
      taskRef:
        name: logger
      params:
        - name: text
          value: "Cleaning up the table"
EOF
```

<br/>

```
$ tkn pipeline start make-your-bets --showlog
```

<br/>

```
PipelineRun started: make-your-bets-run-hr8zr
Waiting for logs to be available...

[check : add-card] New hand value is 21
[check : add-card] You won
[check : add-card] + HAND=17
[check : add-card] + NEWCARD=4
[check : add-card] + NEWHAND=21
[check : add-card] + echo 'New hand value is 21'
[check : add-card] + (( 21 > 21 ))
[check : add-card] + echo 'You won'
[check : add-card] + exit 0

[clean : log] ++ date '+%d/%m/%Y %T'
[clean : log] + DATE='17/10/2021 14:43:54'
[clean : log] + echo '[17/10/2021' '14:43:54]' - Cleaning up the table
[clean : log] [17/10/2021 14:43:54] - Cleaning up the table
```
