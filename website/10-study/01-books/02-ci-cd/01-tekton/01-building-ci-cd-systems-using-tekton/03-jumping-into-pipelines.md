---
layout: page
title: Building CI/CD Systems Using Tekton - Jumping into Pipelines
description: Building CI/CD Systems Using Tekton - Jumping into Pipelines
keywords: books, ci-cd, tekton, Jumping into Pipelines
permalink: /study/books/ci-cd/tekton/building-ci-cd-systems-using-tekton/jumping-into-pipelines/
---

# Chapter 5. Jumping into Pipelines

<br/>

### Building your first pipeline

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
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
$ cat << 'EOF' | kubectl apply -f -
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
$ cat << 'EOF' | kubectl apply -f -
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

first-task д.б. создана.

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
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
$ cat << 'EOF' | kubectl apply -f -
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

```
PipelineRun started: parametrized-dice-roll-run-spgwz
Waiting for logs to be available...
[first : unnamed-0] Hello from first task

[roll : greetings] Rolling 6-sided dice

[roll : generate-random-number] 4
```

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
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
$ cat << 'EOF' | kubectl apply -f -
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
PipelineRun started: parametrized-dice-roll-run-mqmwp
Waiting for logs to be available...
[first : unnamed-0] Hello from first task

[roll : greetings] Rolling 12-sided dice

[roll : generate-random-number] 7
```

<br/>

```
$ tkn pipeline start parametrized-dice-roll --use-param-defaults --showlog
```

<br/>

### Reusing tasks in the context of a pipeline

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
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
$ cat << 'EOF' | kubectl apply -f -
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

```
PipelineRun started: task-reuse-run-27qr6
Waiting for logs to be available...
[say-hello : log] ++ date '+%d/%m/%Y %T'
[say-hello : log] + DATE='17/10/2021 14:34:46'
[say-hello : log] + echo '[17/10/2021' '14:34:46]' - Hello
[say-hello : log] [17/10/2021 14:34:46] - Hello
[log-something : log] ++ date '+%d/%m/%Y %T'
[log-something : log] + DATE='17/10/2021 14:34:46'
[log-something : log] + echo '[17/10/2021' '14:34:46]' - Something else being logged
[log-something : log] [17/10/2021 14:34:46] - Something else being logged
```

<br/>

### Ordering tasks within pipelines

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
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
$ cat << 'EOF' | kubectl apply -f -
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

```
PipelineRun started: ordered-tasks-run-6lxzh
Waiting for logs to be available...
[first : init] [17/10/2021 14:35:36] - Task A Started


[first : log] [17/10/2021 14:35:40] - Task A Completed

[second : init] [17/10/2021 14:35:49] - Task B Started

[third : init] [17/10/2021 14:35:50] - Task C Started

[second : log] [17/10/2021 14:35:50] - Task B Completed



[third : log] [17/10/2021 14:35:54] - Task C Completed

[fourth : init] [17/10/2021 14:36:00] - Task D Started


[fourth : log] [17/10/2021 14:36:02] - Task D Completed


```

<br/>

### Using task results in pipelines

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
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
$ cat << 'EOF' | kubectl apply -f -
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

```
? Value for param `sides` of type `string`? (Default is `6`) 6
PipelineRun started: results-run-ln86k
Waiting for logs to be available...
[intro : log] ++ date '+%d/%m/%Y %T'
[intro : log] [17/10/2021 14:36:44] - Preparing to roll the 6-sided dice
[intro : log] + DATE='17/10/2021 14:36:44'
[intro : log] + echo '[17/10/2021' '14:36:44]' - Preparing to roll the 6-sided dice

[roll : generate-random-number] Dice rolled

[result : log] ++ date '+%d/%m/%Y %T'
[result : log] [17/10/2021 14:36:55] - Result from dice roll was 0
[result : log] + DATE='17/10/2021 14:36:55'
[result : log] + echo '[17/10/2021' '14:36:55]' - Result from dice roll was 0

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

<br/>

## Assessments

<br/>

### Back to the basics

<br/>

Task logger должна быть создана.

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: back-to-basics
spec:
  params:
    - name: who
      default: "World"
      type: string
      description: Who should we say hello to?
  tasks:
    - name: say-hello
      params:
        - name: text
          value: Hello $(params.who)
      taskRef:
        name: logger
EOF
```

<br/>

```
$ tkn pipeline start back-to-basics --showlog
```

<br/>

```
? Value for param `who` of type `string`? (Default is `World`) World
PipelineRun started: back-to-basics-run-7f8rf
Waiting for logs to be available...
[say-hello : log] ++ date '+%d/%m/%Y %T'
[say-hello : log] + DATE='17/10/2021 14:26:48'
[say-hello : log] + echo '[17/10/2021' '14:26:48]' - Hello World
[say-hello : log] [17/10/2021 14:26:48] - Hello World
```

<br/>

### Counting files in a repo

<br/>

Task logger должна быть создана.

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: clone-and-count
spec:
  params:
    - name: repo
      type: string
  results:
    - name: file-count
      description: Number of files
  steps:
    - name: clone-and-ls
      image: alpine/git
      script: |
        git clone $(params.repo) .
        ls | wc -l > $(results.file-count.path)
---
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: count-files
spec:
  params:
    - name: repo-to-analyze
  tasks:
    - name: get-list
      taskRef:
        name: clone-and-count
      params:
        - name: repo
          value: $(params.repo-to-analyze)
    - name: output-count
      taskRef:
        name: logger
      params:
        - name: text
          value: "Number of files in $(params.repo-to-analyze): $(tasks.get-list.results.file-count)"
      runAfter:
        - get-list
EOF
```

<br/>

```
$ tkn pipeline start count-files --showlog
```

<br/>

```
? Value for param `repo-to-analyze` of type `string`? https://github.com/PacktPublishing/Building-CI-CD-systems-using-Tekton
PipelineRun started: count-files-run-nmcvf
Waiting for logs to be available...
[get-list : clone-and-ls] + git clone https://github.com/PacktPublishing/Building-CI-CD-systems-using-Tekton .
[get-list : clone-and-ls] Cloning into '.'...
[get-list : clone-and-ls] + ls
[get-list : clone-and-ls] + wc -l

[output-count : log] ++ date '+%d/%m/%Y %T'
[output-count : log] + DATE='17/10/2021 14:26:08'
[output-count : log] [17/10/2021 14:26:08] - Number of files in https://github.com/PacktPublishing/Building-CI-CD-systems-using-Tekton: 11
[output-count : log] + echo '[17/10/2021' '14:26:08]' - Number of files in https://github.com/PacktPublishing/Building-CI-CD-systems-using-Tekton: 11
```

<br/>

### Weather services

<br/>

Task logger должна быть создана.

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: weather
spec:
  params:
    - name: city
      type: string
  results:
    - name: weather
      description: JSON object with weather definition
  steps:
    - name: get-weather
      image: registry.access.redhat.com/ubi8/ubi
      script: |
        curl wttr.in/$(params.city)?format=4 -o $(results.weather.path)
---
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: weather-extract
spec:
  results:
    - name: temperature
      description: Current temperature
  params:
    - name: weather-data
      type: string
  steps:
    - name: extract-data
      image: registry.access.redhat.com/ubi8/ubi
      script: |
        echo "$(params.weather-data)" | awk '{print $3}' > $(results.temperature.path)
---
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: weather
spec:
  params:
    - name: city
      type: string
      default: Ottawa
  tasks:
    - name: get-weather
      params:
        - name: city
          value: $(params.city)
      taskRef:
        name: weather
    - name: extract-data
      params:
        - name: weather-data
          value: $(tasks.get-weather.results.weather)
      taskRef:
        name: weather-extract
      runAfter:
        - get-weather
    - name: current-temperature
      params:
        - name: text
          value: Current temperature in $(params.city) is $(tasks.extract-data.results.temperature)
      taskRef:
        name: logger
      runAfter:
        - extract-data
EOF
```

<br/>

```
$ tkn pipeline start weather --showlog
```

<br/>

```
? Value for param `city` of type `string`? (Default is `Ottawa`) Ottawa
PipelineRun started: weather-run-cv4sk
Waiting for logs to be available...
[get-weather : get-weather] + curl 'wttr.in/Ottawa?format=4' -o /tekton/results/weather
[get-weather : get-weather]   % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
[get-weather : get-weather]                                  Dload  Upload   Total   Spent    Left  Speed
100    46  100    46    0     0    282      0 --:--:-- --:--:-- --:--:--   282

[extract-data : extract-data] + echo 'Ottawa: ⛅️  🌡️+8°C 🌬️→11km/h
[extract-data : extract-data] + awk '{print $3}'
[extract-data : extract-data] '

[current-temperature : log] ++ date '+%d/%m/%Y %T'
[current-temperature : log] + DATE='17/10/2021 14:29:20'
[current-temperature : log] + echo '[17/10/2021' '14:29:20]' - Current temperature in Ottawa is $'\360\237\214\241\357\270\217+8\302\260C'
[current-temperature : log] [17/10/2021 14:29:20] - Current temperature in Ottawa is 🌡️+8°C

```
