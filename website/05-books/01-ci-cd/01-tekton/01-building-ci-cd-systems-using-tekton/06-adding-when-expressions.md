---
layout: page
title: Building CI/CD Systems Using Tekton - Adding when Expressions
description: Building CI/CD Systems Using Tekton - Adding when Expressions
keywords: books, ci-cd, tekton, Adding when Expressions
permalink: /books/ci-cd/tekton/building-ci-cd-systems-using-tekton/adding-when-expressions/
---

# Chapter 8. Adding when Expressions

<br/>

### Using when expressions with parameters

<br/>

```
$ vi ~/tmp/logger.yaml
```

```yaml
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
```

<br/>

```
$ kubectl create -f ~/tmp/logger.yaml
```

<br/>

```
$ vi ~/tmp/guess.yaml
```

<br/>

```yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
    name: guess-game
spec:
    params:
        - name: number
          description: Pick a number
          type: string
    tasks:
        - name: win
          params:
              - name: text
                value: You win
          taskRef:
              name: logger
          when:
              - input: $(params.number)
                operator: in
                values: ['3']
```

<br/>

```
$ kubectl create -f ~/tmp/guess.yaml
```

<br/>

```
$ tkn pipeline start guess-game --showlog
```

<br/>

```
? Value for param `number` of type `string`? 3
```

В общем только на 3 выводит лог.

<br/>

### Using the notin operator

<br/>

```
$ vi ~/tmp/guess-notin.yaml
```

<br/>

```yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
    name: guess-game-notin
spec:
    params:
        - name: number
          description: Pick a number
          type: string
    tasks:
        - name: win
          params:
              - name: text
                value: You win
          taskRef:
              name: logger
          when:
              - input: $(params.number)
                operator: in
                values: ['3']
        - name: lose
          params:
              - name: text
                value: You lose
          taskRef:
              name: logger
          when:
              - input: $(params.number)
                operator: notin
                values: ['3']
```

<br/>

```
$ kubectl create -f ~/tmp/guess-notin.yaml
```

<br/>

```
$ tkn pipeline start guess-game-notin --showlog
```

Теперь 3 и не 3.

<br/>

### Using when expressions with results

<br/>

```
$ vi ~/tmp/guess-result.yaml
```

<br/>

```yaml
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
    name: random-number-generator
spec:
    results:
        - name: random-number
          description: random number
    steps:
        - name: generate-number
          image: registry.access.redhat.com/ubi8/ubi-minimal
          script: |
              NUMBER=$((1 + $RANDOM % 3))
              echo Random number picked, result is $NUMBER
              echo -n $NUMBER > $(results.random-number.path)
---
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
    name: guess-result
spec:
    tasks:
        - name: generate
          taskRef:
              name: random-number-generator
        - name: win
          params:
              - name: text
                value: You win
          taskRef:
              name: logger
          when:
              - input: $(tasks.generate.results.random-number)
                operator: in
                values: ['3']
          runAfter:
              - generate
        - name: lose
          params:
              - name: text
                value: You lose
          taskRef:
              name: logger
          when:
              - input: $(tasks.generate.results.random-number)
                operator: notin
                values: ['3']
          runAfter:
              - generate
```

<br/>

```
$ kubectl create -f ~/tmp/guess-result.yaml
```

<br/>

```
$ tkn pipeline start guess-result --showlog
```
