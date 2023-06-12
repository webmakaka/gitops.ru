---
layout: page
title: GitOps Cookbook - Cloud Native CI/CD - Tekton - Create a Tekton Pipeline to Build and Deploy an App to Kubernetes
description: GitOps Cookbook - Cloud Native CI/CD - Tekton - Create a Tekton Pipeline to Build and Deploy an App to Kubernetes
keywords: GitOps Cookbook - Cloud Native CI/CD, Tekton, Create a Tekton Pipeline to Build and Deploy an App to Kubernetes
permalink: /books/gitops/gitops-cookbook/cloud-native-cicd/tekton/create-a-tekton-pipeline-to-build-and-deploy-an-app-to-kubernetes/
---

<br/>

# [Book] [FAIL!] GitOps Cookbook: 06. Cloud Native CI/CD: Tekton: 6.7 Create a Tekton Pipeline to Build and Deploy an App to Kubernetes

Наверное, шаги из предыдущего параграфа тоже нужны.

<br/>

```
$ {
    export REGISTRY_SERVER=https://index.docker.io/v1/
    export REGISTRY_USER=webmakaka
    export REGISTRY_PASSWORD=webmakaka-password
    export EMAIL=webmakaka-email@mail.ru

    echo ${REGISTRY_SERVER}
    echo ${REGISTRY_USER}
    echo ${REGISTRY_PASSWORD}
    echo ${EMAIL}
}
```

<br/>

```
$ kubectl create secret docker-registry container-registry-secret \
    --docker-server=${REGISTRY_SERVER} \
    --docker-username=${REGISTRY_USER} \
    --docker-password=${REGISTRY_PASSWORD} \
    --docker-email=${EMAIL}
```

<br/>

```
$ kubectl create serviceaccount tekton-deployer-sa
```

<br/>

```
$ kubectl patch serviceaccount tekton-deployer-sa \
-p '{"secrets": [{"name": "container-registry-secret"}]}'
```

<br/>

### Пример 1

<br/>

Создать task build-push-app. Код выше.

<br/>

```yaml
$ cat << 'EOF' | kubectl create -f -
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: tekton-greeter-pipeline
spec:
  params:
    - name: GIT_REPO
      type: string
    - name: GIT_REF
      type: string
    - name : DESTINATION_IMAGE
      type: string
    - name : SCRIPT
      type: string
  tasks:
    - name: build-push-app
      taskRef:
        name: build-push-app
      params:
        - name: url
          value: "$(params.GIT_REPO)"
        - name: revision
          value: "$(params.GIT_REF)"
        - name: destinationImage
          value: "$(params.DESTINATION_IMAGE)"
      workspaces:
        - name: source
    - name: deploy-app
      taskRef:
        name: kubectl
      params:
        - name: SCRIPT
          value: "$(params.SCRIPT)"
      workspaces:
        - name: source
      runAfter:
        - build-push-app
  workspaces:
    - name: source
EOF
```

<br/>

```
$ export NEW_IMAGE=webmakaka/tekton-greeter:latest
```

<br/>

```yaml
$ envsubst << 'EOF' | cat | kubectl create -f -
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  generateName: tekton-greeter-pipeline-run-
spec:
  params:
  - name: GIT_REPO
    value: https://github.com/gitops-cookbook/tekton-tutorial-greeter.git
  - name: GIT_REF
    value: "master"
  - name: DESTINATION_IMAGE
    value: ${NEW_IMAGE}
  - name: SCRIPT
    value: |
      kubectl create deploy tekton-greeter --image=${NEW_IMAGE}
  pipelineRef:
    name: tekton-greeter-pipeline
  workspaces:
    - name: source
      emptyDir: {}
EOF
```

<br/>

```
// FAIL!
$ tkn pipelinerun ls
NAME                                STARTED         DURATION   STATUS
tekton-greeter-pipeline-run-6pdpx       2 minutes ago    2m11s      Failed
```

<br/>

```
$ tkn pipelinerun logs tekton-greeter-pipeline-run-pf8x9
```

<br/>

```
[build-push-app : build-and-push-image] Error: pushing image "webmakaka/tekton-greeter:latest" to "docker://webmakaka/tekton-greeter:latest": writing blob: initiating layer upload to /v2/webmakaka/tekton-greeter/blobs/uploads/ in registry-1.docker.io: requested access to the resource is denied
```

<br/>

### Пример 2

<br/>

https://hub.tekton.dev

<br/>

```
$ tkn hub install task git-clone
$ tkn hub install task maven
$ tkn hub install task buildah
$ tkn hub install task kubernetes-actions
```

<br/>

```
$ kubectl get tasks
NAME                 AGE
buildah              66s
git-clone            82s
kubectl              18m
kubernetes-actions   62s
maven                70s
```

<br/>

```yaml
$ cat << 'EOF' | kubectl create -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-source-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
EOF
```

<br/>

```
$ kubectl get pvc
NAME             STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
app-source-pvc   Bound    pvc-c675c620-2268-43a3-835d-8a99743edf69   1Gi        RWO            standard       5s
```

<br/>

```
$ export NEW_IMAGE=webmakaka/tekton-greeter:latest
```

<br/>

```yaml
$ envsubst << 'EOF' | cat | kubectl create -f -
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: tekton-greeter-pipeline-hub
spec:
  params:
  - default: https://github.com/gitops-cookbook/tekton-tutorial-greeter.git
    name: GIT_REPO
    type: string
  - default: master
    name: GIT_REF
    type: string
  - default: ${NEW_IMAGE}
    name: DESTINATION_IMAGE
    type: string
  - default: kubectl create deploy tekton-greeter --image=${NEW_IMAGE}
    name: SCRIPT
    type: string
  - default: ./Dockerfile
    name: CONTEXT_DIR
    type: string
  - default: .
    name: IMAGE_DOCKERFILE
    type: string
  - default: .
    name: IMAGE_CONTEXT_DIR
    type: string
  tasks:
  - name: fetch-repo
    params:
    - name: url
      value: $(params.GIT_REPO)
    - name: revision
      value: $(params.GIT_REF)
    - name: deleteExisting
      value: "true"
    - name: verbose
      value: "true"
    taskRef:
      kind: Task
      name: git-clone
    workspaces:
    - name: output
      workspace: app-source
  - name: build-app
    params:
    - name: GOALS
      value:
      - -DskipTests
      - clean
      - package
    - name: CONTEXT_DIR
      value: $(params.CONTEXT_DIR)
    runAfter:
    - fetch-repo
    taskRef:
      kind: Task
      name: maven
    workspaces:
    - name: maven-settings
      workspace: maven-settings
    - name: source
      workspace: app-source
  - name: build-push-image
    params:
    - name: IMAGE
      value: $(params.DESTINATION_IMAGE)
    - name: DOCKERFILE
      value: $(params.IMAGE_DOCKERFILE)
    - name: CONTEXT
      value: $(params.IMAGE_CONTEXT_DIR)
    runAfter:
    - build-app
    taskRef:
      kind: Task
      name: buildah
    workspaces:
    - name: source
      workspace: app-source
  - name: deploy
    params:
    - name: script
      value: $(params.SCRIPT)
    runAfter:
    - build-push-image
    taskRef:
      kind: Task
      name: kubernetes-actions
  workspaces:
  - name: app-source
  - name: maven-settings
EOF
```

<br/>

```
// Не удалось переменной задать image
// OK!
$ tkn pipeline start tekton-greeter-pipeline-hub \
    --serviceaccount='tekton-deployer-sa' \
    --param GIT_REPO='https://github.com/gitops-cookbook/tekton-tutorial-greeter.git' \
    --param GIT_REF='master' \
    --param CONTEXT_DIR='quarkus' \
    --param DESTINATION_IMAGE=webmakaka/tekton-greeter:latest \
    --param IMAGE_DOCKERFILE='quarkus/Dockerfile' \
    --param IMAGE_CONTEXT_DIR='quarkus' \
    --param SCRIPT='kubectl create deploy tekton-greeter --image=webmakaka/tekton-greeter:latest' \
    --workspace name=app-source,claimName=app-source-pvc \
    --workspace name=maven-settings,emptyDir="" \
    --use-param-defaults \
    --showlog
```

<br/>

```
$ tkn pipelinerun ls
NAME                                    STARTED        DURATION   STATUS
tekton-greeter-pipeline-hub-run-42pdx   2 minutes ago    1m39s      Succeeded
```

<br/>

```
$ tkn pipelinerun logs tekton-greeter-pipeline-hub-run-vtc84
```

<br/>

**Посмотреть результаты в UI**

```
$ kubectl --namespace tekton-pipelines port-forward svc/tekton-dashboard 8080:9097
```

<br/>

```
$ localhost:8080 -> PipelineRuns
```
