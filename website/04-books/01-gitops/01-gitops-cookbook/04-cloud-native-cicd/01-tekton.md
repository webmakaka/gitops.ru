---
layout: page
title: GitOps Cookbook - Cloud Native CI/CD - Tekton
description: GitOps Cookbook - Cloud Native CI/CD - Tekton
keywords: GitOps Cookbook - Cloud Native CI/CD, Tekton
permalink: /books/gitops/gitops-cookbook/cloud-native-cicd/tekton/
---

<br/>

# [Book] GitOps Cookbook: 06. Cloud Native CI/CD: Tekton

<br/>

**Делаю:**  
29.05.2023

<br/>

[Поставил Tekton + Triggers](/tools/containers/kubernetes/tools/ci-cd/tekton/)

<br/>

### 6.2 Create a Hello World Task

<br/>

[Повтор Building your first task](/books/ci-cd/tekton/building-ci-cd-systems-using-tekton/stepping-into-tasks/)

<br/>

### [OK!] 6.3 Create a Task to Compile and Package an App from Git

<br/>

```yaml
$ cat << 'EOF' | kubectl create -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: build-app
spec:
  workspaces:
    - name: source
      description: The git repo will be cloned onto the volume backing this work space
  params:
    - name: contextDir
      description: the context dir within source
      default: quarkus
    - name: tlsVerify
      description: tls verify
      type: string
      default: "false"
    - name: url
      default: https://github.com/gitops-cookbook/tekton-tutorial-greeter.git
    - name: revision
      default: master
    - name: subdirectory
      default: ""
    - name: sslVerify
      description: defines if http.sslVerify should be set to true or false in the global git config
      type: string
      default: "false"
  steps:
    - image: 'gcr.io/tekton-releases/github.com/tektoncd/pipeline/cmd/git-init:v0.21.0'
      name: clone
      resources: {}
      script: |
          CHECKOUT_DIR="$(workspaces.source.path)/$(params.subdirectory)"
          cleandir() {
          # Delete any existing contents of the repo directory if it exists.
          #
          # We don't just "rm -rf $CHECKOUT_DIR" because $CHECKOUT_DIR might be "/"
          # or the root of a mounted volume.
          if [[ -d "$CHECKOUT_DIR" ]] ; then
          # Delete non-hidden files and directories
          rm -rf "$CHECKOUT_DIR"/*
          # Delete files and directories starting with . but excluding ..
          rm -rf "$CHECKOUT_DIR"/.[!.]*
          # Delete files and directories starting with .. plus any other character
          rm -rf "$CHECKOUT_DIR"/..?*
          fi
          }
          /ko-app/git-init \
          -url "$(params.url)" \
          -revision "$(params.revision)" \
          -path "$CHECKOUT_DIR" \
          -sslVerify="$(params.sslVerify)"
          cd "$CHECKOUT_DIR"
          RESULT_SHA="$(git rev-parse HEAD)"
    - name: build-sources
      image: gcr.io/cloud-builders/mvn
      command:
        - mvn
      args:
        - -DskipTests
        - clean
        - install
      env:
        - name: user.home
          value: /home/tekton
      workingDir: "/workspace/source/$(params.contextDir)"
EOF
```

<br/>

```yaml
$ cat << 'EOF' | kubectl create -f -
apiVersion: tekton.dev/v1beta1
kind: TaskRun
metadata:
  generateName: build-app-run-
  labels:
    app.kubernetes.io/managed-by: tekton-pipelines
    tekton.dev/task: build-app
spec:
  params:
  - name: contextDir
    value: quarkus
  - name: revision
    value: master
  - name: sslVerify
    value: "false"
  - name: subdirectory
    value: ""
  - name: tlsVerify
    value: "false"
  - name: url
    value: https://github.com/gitops-cookbook/tekton-tutorial-greeter.git
  taskRef:
    kind: Task
    name: build-app
  workspaces:
  - emptyDir: {}
    name: source
EOF
```

<br/>

```
$ kubectl get tasks
build-app   36s
```

<br/>

```
$ tkn taskrun ls
NAME                  STARTED          DURATION   STATUS
build-app-run-f5ctk   3 minutes ago    1m38s      Succeeded
```

<br/>

```
$ tkn taskrun logs build-app-run-f5ctk -f
[build-sources] [INFO] ------------------------------------------------------------------------
[build-sources] [INFO] BUILD SUCCESS
[build-sources] [INFO] ------------------------------------------------------------------------
[build-sources] [INFO] Total time:  47.200 s
[build-sources] [INFO] Finished at: 2023-05-29T10:18:22Z
[build-sources] [INFO] ------------------------------------------------------------------------
```

<br/>

### [OK!] 6.4 Create a Task to Compile and Package an App from Private Git

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

<br/>

### [OK!] 6.5 Containerize an Application Using a Tekton Task and Buildah

<br/>

// Насколько понимаю, email нужен

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
$ kubectl get secrets
```

<br/>

```
$ kubectl create serviceaccount tekton-registry-sa
```

<br/>

```
$ kubectl patch serviceaccount tekton-registry-sa \
-p '{"secrets": [{"name": "container-registry-secret"}]}'
```

<br/>

```yaml
$ cat << 'EOF' | kubectl create -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: build-push-app
spec:
  workspaces:
    - name: source
      description: The git repo will be cloned onto the volume backing this work space
  params:
    - name: contextDir
      description: the context dir within source
      default: quarkus
    - name: tlsVerify
      description: tls verify
      type: string
      default: "false"
    - name: url
      default: https://github.com/gitops-cookbook/tekton-tutorial-greeter.git
    - name: revision
      default: master
    - name: subdirectory
      default: ""
    - name: sslVerify
      description: defines if http.sslVerify should be set to true or false in the global git config
      type: string
      default: "false"
    - name: storageDriver
      type: string
      description: Storage driver
      default: vfs
    - name: destinationImage
      description: the fully qualified image name
      default: ""
  steps:
    - image: 'gcr.io/tekton-releases/github.com/tektoncd/pipeline/cmd/git-init:v0.21.0'
      name: clone
      resources: {}
      script: |
          CHECKOUT_DIR="$(workspaces.source.path)/$(params.subdirectory)"
          cleandir() {
          # Delete any existing contents of the repo directory if it exists.
          #
          # We don't just "rm -rf $CHECKOUT_DIR" because $CHECKOUT_DIR might be "/"
          # or the root of a mounted volume.
          if [[ -d "$CHECKOUT_DIR" ]] ; then
          # Delete non-hidden files and directories
          rm -rf "$CHECKOUT_DIR"/*
          # Delete files and directories starting with . but excluding ..
          rm -rf "$CHECKOUT_DIR"/.[!.]*
          # Delete files and directories starting with .. plus any other character
          rm -rf "$CHECKOUT_DIR"/..?*
          fi
          }
          /ko-app/git-init \
          -url "$(params.url)" \
          -revision "$(params.revision)" \
          -path "$CHECKOUT_DIR" \
          -sslVerify="$(params.sslVerify)"
          cd "$CHECKOUT_DIR"
          RESULT_SHA="$(git rev-parse HEAD)"
    - name: build-sources
      image: gcr.io/cloud-builders/mvn
      command:
        - mvn
      args:
        - -DskipTests
        - clean
        - install
      env:
        - name: user.home
          value: /home/tekton
      workingDir: "/workspace/source/$(params.contextDir)"
    - name: build-and-push-image
      image: quay.io/buildah/stable
      script: |
          #!/usr/bin/env bash
          buildah --storage-driver=$STORAGE_DRIVER bud --layers -t $DESTINATION_IMAGE $CONTEXT_DIR
          buildah --storage-driver=$STORAGE_DRIVER push $DESTINATION_IMAGE docker://$DESTINATION_IMAGE
      env:
        - name: DESTINATION_IMAGE
          value: "$(params.destinationImage)"
        - name: CONTEXT_DIR
          value: "/workspace/source/$(params.contextDir)"
        - name: STORAGE_DRIVER
          value: "$(params.storageDriver)"
      workingDir: "/workspace/source/$(params.contextDir)"
      volumeMounts:
        - name: varlibc
          mountPath: /var/lib/containers
  volumes:
    - name: varlibc
      emptyDir: {}
EOF
```

<br/>

```
// OK!
$ tkn task start build-push-app \
  --serviceaccount='tekton-registry-sa' \
  --param url='https://github.com/gitops-cookbook/tekton-tutorial-greeter.git' \
  --param destinationImage='webmakaka/tekton-greeter:latest' \
  --param contextDir='quarkus' \
  --workspace name=source,emptyDir="" \
  --use-param-defaults \
  --showlog
```

<br/>

```
[build-sources] [INFO] ------------------------------------------------------------------------
[build-sources] [INFO] BUILD SUCCESS
[build-sources] [INFO] ------------------------------------------------------------------------
[build-sources] [INFO] Total time:  36.180 s
[build-sources] [INFO] Finished at: 2023-05-29T14:01:04Z
[build-sources] [INFO] ------------------------------------------------------------------------

[build-and-push-image] STEP 1/2: FROM registry.access.redhat.com/ubi8/openjdk-11
[build-and-push-image] Trying to pull registry.access.redhat.com/ubi8/openjdk-11:latest...
[build-and-push-image] Getting image source signatures
[build-and-push-image] Checking if image destination supports signatures
[build-and-push-image] Copying blob sha256:d09aca24592b99820eb623c3a56914ab82562e5a4e37aa67ece0402d832e3100
[build-and-push-image] Copying blob sha256:06f86e50a0b74ff9eb161a7d781228877c90e8ff57e9689e8cb8b0f092a2a9f9
[build-and-push-image] Copying config sha256:d1ce871371c268991ea2f4c4dd5b5dcd972f9a68bc55f48b320afe6fa43482b9
[build-and-push-image] Writing manifest to image destination
[build-and-push-image] Storing signatures
[build-and-push-image] STEP 2/2: COPY target/quarkus-app /deployments/
[build-and-push-image] COMMIT webmakaka/tekton-greeter:latest
[build-and-push-image] --> a3ac36b0d6c9
[build-and-push-image] Successfully tagged localhost/webmakaka/tekton-greeter:latest
[build-and-push-image] a3ac36b0d6c97b37e6bcb3193637e3e81777ec0fac5cd60fa89dcf4d7625b11f
[build-and-push-image] Getting image source signatures
[build-and-push-image] Copying blob sha256:a02aa3b0d74d2d11c65b91ed11d8e477d0dcfaa532ed44d3ea851fef827e399b
[build-and-push-image] Copying blob sha256:969795712c492f0c43031ce89dfb3d6ca2c08221fc28fb4479c7e0a370af7342
[build-and-push-image] Copying blob sha256:65e05838a57a7ebfcc49994779f4a29e3548b78e9d12542320d0fbc79dc555c3
[build-and-push-image] Copying config sha256:a3ac36b0d6c97b37e6bcb3193637e3e81777ec0fac5cd60fa89dcf4d7625b11f
[build-and-push-image] Writing manifest to image destination
[build-and-push-image] Storing signatures
```

<br/>

### [OK!] 6.6 Deploy an Application to Kubernetes Using a Tekton Task

<br/>

**Делаю:**  
30.05.2023

<br/>

```yaml
$ cat << 'EOF' | kubectl create -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: kubectl
spec:
  params:
    - name: SCRIPT
      description: The kubectl CLI arguments to run
      type: string
      default: "kubectl help"
  steps:
    - name: oc
      image: quay.io/openshift/origin-cli:latest
      script: |
        #!/usr/bin/env bash
        $(params.SCRIPT)
EOF
```

<br/>

```
$ kubectl create serviceaccount tekton-deployer-sa
```

<br/>

Define a Role named pipeline-role for the ServiceAccount

<br/>

```yaml
$ cat << 'EOF' | kubectl create -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: task-role
rules:
  - apiGroups:
      - ""
    resources:
      - pods
      - services
      - endpoints
      - configmaps
      - secrets
    verbs:
      - "*"
  - apiGroups:
      - apps
    resources:
      - deployments
      - replicasets
    verbs:
      - "*"
  - apiGroups:
      - ""
    resources:
      - pods
    verbs:
      - get
  - apiGroups:
      - apps
    resources:
      - replicasets
    verbs:
      - get
EOF
```

<br/>

Bind the Role to the ServiceAccount :

```yaml
$ cat << 'EOF' | kubectl create -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: task-role-binding
roleRef:
  kind: Role
  name: task-role
  apiGroup: rbac.authorization.k8s.io
subjects:
  - kind: ServiceAccount
    name: tekton-deployer-sa
EOF
```

<br/>

Define a TaskRun

<br/>

```yaml
$ cat << 'EOF' | kubectl create -f -
apiVersion: tekton.dev/v1beta1
kind: TaskRun
metadata:
  name: kubectl-taskrun
spec:
  serviceAccountName: tekton-deployer-sa
  taskRef:
    name: kubectl
  params:
    - name: SCRIPT
      value: |
        kubectl create deploy tekton-greeter --image=quay.io/gitops-cookbook/tekton-greeter:latest
EOF
```

<br/>

```
$ tkn taskrun logs kubectl-taskrun -f
```

<br/>

```
$ kubectl get deploy
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
tekton-greeter   1/1     1            1           50s
```

<br/>

```
$ kubectl expose deploy/tekton-greeter --port 8080
$ kubectl port-forward svc/tekton-greeter 8080:8080
```

<br/>

```
curl localhost:8080
Meeow!! from Tekton 😺🚀⏎
```

<br/>

### [FAIL!] 6.7 Create a Tekton Pipeline to Build and Deploy an App to Kubernetes

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
