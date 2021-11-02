---
layout: page
title: Building CI/CD Systems Using Tekton - Building a Deployment Pipeline
description: Building CI/CD Systems Using Tekton - Building a Deployment Pipeline
keywords: books, ci-cd, tekton, Building a Deployment Pipeline
permalink: /study/books/ci-cd/tekton/building-ci-cd-systems-using-tekton/building-a-deployment-pipeline/
---

# Chapter 13. Building a Deployment Pipeline

<br/>

Let's think about what operations are needed every time you perform a commit on your source code:

<br/>

1. Clone the repository.
2. Install the required libraries.
3. Test the code.
4. Lint the code.
5. Build and push the image.
6. Deploy the application.

<br/>

### Using the task catalog

https://hub.tekton.dev/

```
$ tkn hub install task git-clone
$ tkn hub install task npm
$ tkn hub install task kubernetes-actions
```

<br/>

### Adding an additional task

Можно попробовать использовать task "docker build task".
Но требуется обращение к Docker демону по сокету (или как-то так) и не работает во всех окружения.

Предлагают использовать Buildah

https://buildah.io/

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: build-push
spec:
  params:
    - name: image
    - name: username
    - name: password
  workspaces:
    - name: source
  steps:
    - name: build-image
      image: quay.io/buildah/stable:v1.18.0
      securityContext:
        privileged: true
      script: |
        cd $(workspaces.source.path)
        buildah bud --layers -t $(params.image) .
        buildah login -u $(params.username) -p $(params.password) docker.io
        buildah push $(params.image)
EOF
```

<br/>

### Creating the pipeline

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: tekton-deploy
spec:
  params:
    - name: repo-url
    - name: deployment-name
    - name: image
    - name: docker-username
    - name: docker-password
  workspaces:
    - name: source
  tasks:
    - name: clone
      taskRef:
        name: git-clone
      params:
        - name: url
          value: $(params.repo-url)
      workspaces:
        - name: output
          workspace: source
    - name: install
      taskRef:
        name: npm
      params:
        - name: ARGS
          value:
            - install
      workspaces:
        - name: source
          workspace: source
      runAfter:
        - clone
    - name: lint
      taskRef:
        name: npm
      params:
        - name: ARGS
          value:
            - run
            - lint
      workspaces:
        - name: source
          workspace: source
      runAfter:
        - install
    - name: test
      taskRef:
        name: npm
      params:
        - name: ARGS
          value:
            - run
            - test
      workspaces:
        - name: source
          workspace: source
      runAfter:
        - install
    - name: build-push
      taskRef:
        name: build-push
      params:
        - name: image
          value: $(params.image)
        - name: username
          value: $(params.docker-username)
        - name: password
          value: $(params.docker-password)
      workspaces:
        - name: source
          workspace: source
      runAfter:
        - test
        - lint
    - name: deploy
      taskRef:
        name: kubernetes-actions
      params:
        - name: args
          value:
            - rollout
            - restart
            - deployment/$(params.deployment-name)
      runAfter:
        - build-push
EOF
```

<br/>

### Creating the trigger

<br/>

```
$ export TEKTON_SECRET=$(head -c 24 /dev/random | base64)
$ echo ${TEKTON_SECRET}
$ kubectl create secret generic git-secret --from-literal=secretToken=${TEKTON_SECRET}
```

<br/>

**Нужно не забыть заменить <YOUR_USERNAME> и <YOUR_PASSWORD> на свои.**

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
apiVersion: triggers.tekton.dev/v1alpha1
kind: TriggerBinding
metadata:
  name: event-binding
spec:
  params:
    - name: gitrepositoryurl
      value: $(body.repository.url)
---
apiVersion: triggers.tekton.dev/v1alpha1
kind: TriggerTemplate
metadata:
  name: commit-tt
spec:
  params:
  - name: gitrepositoryurl
    description: The git repository url
  resourcetemplates:
  - apiVersion: tekton.dev/v1beta1
    kind: PipelineRun
    metadata:
      generateName: tekton-deploy-
    spec:
      pipelineRef:
        name: tekton-deploy
      params:
        - name: repo-url
          value: $(tt.params.gitrepositoryurl)
        - name: deployment-name
          value: tekton-deployment
        - name: image
          value: <YOUR_USERNAME>/tekton-lab-app
        - name: docker-username
          value: <YOUR_USERNAME>
        - name: docker-password
          value: <YOUR_PASSWORD>
      workspaces:
        - name: source
          volumeClaimTemplate:
            spec:
              accessModes:
              - ReadWriteOnce
              resources:
                requests:
                  storage: 1Gi
---
apiVersion: triggers.tekton.dev/v1alpha1
kind: EventListener
metadata:
  name: listener
spec:
  serviceAccountName: tekton-triggers-example-sa
  triggers:
    - name: trigger
      bindings:
        - ref: event-binding
      template:
        ref: commit-tt
      interceptors:
        - github:
            secretRef:
              secretName: git-secret
              secretKey: secretToken
            eventTypes:
              - push
EOF
```

<br/>

```
$ kubectl port-forward svc/el-listener 8080
```

<br/>

Подключаемся еще 1 терминалом

<br/>

```
$ gcloud cloud-shell ssh
```

<br/>

```
$ cd ~/tmp
$ ./ngrok http 8080
```

<br/>

Github -> MyProject -> Settings -> Webhooks -> Add webhook.

<br/>

• Payload URL: This is your ngrok URL.
• Content type: application/json.
• Secret: Use the secret token you created earlier. You can view your token with the echo $TEKTON_SECRET_TOKEN command.

<br/>

Which events would you like to trigger this webhook?

-   Just the push event

<br/>

Создать

<br/>

Подключаемся еще 1 терминалом

<br/>

```
$ gcloud cloud-shell ssh
```

<br/>

https://github.com/<YOUR_USERNAME>/tekton-book-app/blob/main/server.js

<br/>

Меняем

```
change: "here"
```

<br/>

Меняем

```
change: "the end"
```

<br/>

```
$ tkn pipelineruns ls
NAME                  STARTED         DURATION    STATUS
tekton-deploy-qhpcp   4 minutes ago   2 minutes   Succeeded
```

<br/>

```
// Убеждаемся, что значение профиля установлено
$ echo ${PROFILE}
$ curl $(minikube --profile ${PROFILE} ip)
```

<br/>

**response:**

```
{"message":"Hello","change":"the end"}
```
