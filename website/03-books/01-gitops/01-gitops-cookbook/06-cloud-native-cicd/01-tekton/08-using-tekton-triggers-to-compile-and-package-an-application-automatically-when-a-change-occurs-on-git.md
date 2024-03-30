---
layout: page
title: GitOps Cookbook - Cloud Native CI/CD - Tekton - Using Tekton Triggers to Compile and Package an Application Automatically When a Change Occurs on Git
description: GitOps Cookbook - Cloud Native CI/CD - Tekton - Using Tekton Triggers to Compile and Package an Application Automatically When a Change Occurs on Git
keywords: books, gitops, cloud-native-cicd, tekton, Using Tekton Triggers to Compile and Package an Application Automatically When a Change Occurs on Git
permalink: /books/gitops/gitops-cookbook/cloud-native-cicd/tekton/using-tekton-triggers-to-compile-and-package-an-application-automatically-when-a-change-occurs-on-git/
---

<br/>

# [Book] [OK!] GitOps Cookbook: 06. Cloud Native CI/CD: Tekton: 6.8 Using Tekton Triggers to Compile and Package an Application Automatically When a Change Occurs on Git

<br/>

https://tekton.dev/docs/getting-started/triggers/

<br/>

**Делаю:**  
2024.03.08

<br/>

### Подготовка из предыдущего шага

<br/>

```
$ docker login

***
Login Succeeded
```

<br/>

```
REGISTRY_USER=<your own docker login>
REGISTRY_PASSWORD=<your own docker password>
```

<br/>

```
$ {
    export REGISTRY_SERVER=https://index.docker.io/v1/
    export REGISTRY_USER=webmakaka
    export REGISTRY_PASSWORD=webmakaka-password

    echo ${REGISTRY_SERVER}
    echo ${REGISTRY_USER}
    echo ${REGISTRY_PASSWORD}
}
```

<br/>

```
$ kubectl create secret docker-registry container-registry-secret \
    --docker-server=${REGISTRY_SERVER} \
    --docker-username=${REGISTRY_USER} \
    --docker-password=${REGISTRY_PASSWORD}
```

<br/>

```yaml
$ cat << 'EOF' | kubectl create -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tekton-deployer-sa
secrets:
  - name: container-registry-secret
EOF
```

<br/>

**Define a Role named pipeline-role for the ServiceAccount**

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

**Bind the Role to the ServiceAccount**

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
  - default: webmakaka/tekton-greeter:latest
    name: DESTINATION_IMAGE
    type: string
  - default: kubectl create deploy tekton-greeter --image=webmakaka/tekton-greeter:latest
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
      value: quarkus
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
      value: webmakaka/tekton-greeter:latest
    - name: DOCKERFILE
      value: quarkus/Dockerfile
    - name: CONTEXT
      value: quarkus
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
      value: kubectl create deploy tekton-greeter --image=webmakaka/tekton-greeter:latest
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

### Выполняем шаги текущего параграфа

<br/>

```
// Если нет
$ kubectl get ServiceAccount
$ kubectl get RoleBinding
$ kubectl get ClusterRoleBinding
```

<br/>

```
// Выполнить команду

// This will create a new ServiceAccount named tekton-triggers-sa that has the permissions needed to interact with the Tekton Pipelines component.
$ kubectl apply -f https://raw.githubusercontent.com/tektoncd/triggers/main/examples/rbac.yaml
```

<br/>

```
$ kubectl get pods --namespace tekton-pipelines
NAME                                                READY   STATUS    RESTARTS   AGE
tekton-events-controller-77857f9b75-2dgtj           1/1     Running   0          8m14s
tekton-pipelines-controller-6987c95899-stkt8        1/1     Running   0          8m14s
tekton-pipelines-webhook-7f556bb7d9-6z9jt           1/1     Running   0          8m14s
tekton-triggers-controller-5b6d5f54b7-h6gsm         1/1     Running   0          7m50s
tekton-triggers-core-interceptors-f58696689-gwrpf   1/1     Running   0          7m45s
tekton-triggers-webhook-689688fc54-bvmq5            1/1     Running   0          7m50s
```

<br/>

<!--
```
name: tekton-greeter-pipeline-webhook-$(uid)
```
-->
<!--
В оригинале был прописан:
serviceAccountName: tekton-triggers-example-sa
-->

```yaml
$ cat << 'EOF' | kubectl create -f -
apiVersion: triggers.tekton.dev/v1beta1
kind: TriggerTemplate
metadata:
  name: tekton-greeter-triggertemplate
spec:
  params:
    - name: git-revision
    - name: git-commit-message
    - name: git-repo-url
    - name: git-repo-name
    - name: content-type
    - name: pusher-name
  resourcetemplates:
  - apiVersion: tekton.dev/v1beta1
    kind: PipelineRun
    metadata:
      labels:
        tekton.dev/pipeline: tekton-greeter-pipeline-hub
      name: tekton-greeter-pipeline-webhook-1
    spec:
      serviceAccountName: tekton-deployer-sa
      params:
        - name: GIT_REPO
          value: $(tt.params.git-repo-url)
        - name: GIT_REF
          value: $(tt.params.git-revision)
      pipelineRef:
        name: tekton-greeter-pipeline-hub
      workspaces:
      - name: app-source
        persistentVolumeClaim:
          claimName: app-source-pvc
      - name: maven-settings
        emptyDir: {}
---
apiVersion: triggers.tekton.dev/v1beta1
kind: TriggerBinding
metadata:
  name: tekton-greeter-triggerbinding
spec:
  params:
  - name: git-repo-url
    value: $(body.repository.clone_url)
  - name: git-revision
    value: $(body.after)
---
apiVersion: triggers.tekton.dev/v1beta1
kind: EventListener
metadata:
  name: tekton-greeter-eventlistener
spec:
  serviceAccountName: tekton-triggers-example-sa
  triggers:
  - bindings:
    - ref: tekton-greeter-triggerbinding
    template:
      ref: tekton-greeter-triggertemplate
EOF
```

<br/>

If you are running your Git server outside the cluster (e.g., GitHub or GitLab), you need to expose the Service, for example, with an Ingress. Afterwards you can configure webhooks on your Git server using the EventListener URL associated to your Ingress.

<br/>

We can just simulate the webhook as it would come from the Git server

```
$ kubectl port-forward svc/el-tekton-greeter-eventlistener 8080
```

<br/>

```
$ curl -X POST \
  http://localhost:8080 \
  -H 'Content-Type: application/json' \
  -d '{ "after": "d9291c456db1ce29177b77ffeaa9b71ad80a50e6", "repository": { "clone_url" : "https://github.com/gitops-cookbook/tekton-tutorial-greeter.git" } }' | jq
```

<br/>

```
{
  "eventListener": "tekton-greeter-eventlistener",
  "namespace": "default",
  "eventListenerUID": "210d2e53-d96d-4096-b2d7-4af7239d86b3",
  "eventID": "3ad8301f-43b1-40cb-8a08-b646b99ea4cc"
}
```

<br/>

```
$ tkn pipeline ls
NAME                          AGE             LAST RUN                            STARTED         DURATION   STATUS
tekton-greeter-pipeline-hub   9 minutes ago   tekton-greeter-pipeline-webhook-1   6 minutes ago   5m18s      Failed
```

<br/>

```
$ tkn pipelinerun ls
NAME                                STARTED         DURATION   STATUS
tekton-greeter-pipeline-webhook-1   5 minutes ago   5m18s      Failed
```

<br/>

```
$ tkn pipelinerun logs tekton-greeter-pipeline-webhook-1 -f
```

<br/>

```
$ kubectl get pods
$ kubectl logs tekton-greeter-pipeline-webhook-1-deploy-pod | jq
```

Если запускать повторно, то:

```
Error from server (AlreadyExists): deployments.apps "tekton-greeter" already exists
```
