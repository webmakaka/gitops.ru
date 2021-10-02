---
layout: page
title: Building CI/CD Systems Using Tekton - Sharing Data with Workspaces
description: Building CI/CD Systems Using Tekton - Sharing Data with Workspaces
keywords: books, ci-cd, tekton, Chapter 7. Sharing Data with Workspaces
permalink: /books/ci-cd/tekton/building-ci-cd-systems-using-tekton/sharing-data-with-workspaces/
---

# Chapter 7. Sharing Data with Workspaces

<br/>

Workspaces are shared volumes used to transfer data between the various steps of a task.

<br/>

**Types of volume sources:**

-   emptyDir
-   ConfigMap
-   Secret

<br/>

### Using your first workspace

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: clone-and-list
spec:
  params:
    - name: repo
      type: string
      description: Git repository to be cloned
      default: https://github.com/joellord/handson-tekton
  workspaces:
    - name: source
  steps:
    - name: clone
      image: alpine/git
      workingDir: $(workspaces.source.path)
      command:
        - /bin/sh
      args:
        - '-c'
        - git clone -v $(params.repo) ./source
    - name: list
      image: alpine
      workingDir: $(workspaces.source.path)
      command:
        - /bin/sh
      args:
        - "-c"
        - ls ./source
EOF
```

<br/>

```
$ tkn task start clone-and-list -w name=source,emptyDir="" --showlog
```

<br/>

### Using workspaces with task runs

<br/>

```
$ vi ~/tmp/clone-and-list-tr.yaml
```

```yaml
apiVersion: tekton.dev/v1beta1
kind: TaskRun
metadata:
    generateName: git-clone-tr-
spec:
    workspaces:
        - name: source
          emptyDir: {}
    taskRef:
        name: clone-and-list
```

<br/>

```
$ kubectl create -f ~/tmp/clone-and-list-tr.yaml
```

<br/>

```
$ tkn taskrun logs git-clone-tr-c22wd
```

<br/>

### Adding a workspace to a pipeline

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: clone
spec:
  params:
    - name: repo
      type: string
      description: Git repository to be cloned
      default: https://github.com/joellord/handson-tekton
  workspaces:
    - name: source
  steps:
    - name: clone
      image: alpine/git
      workingDir: $(workspaces.source.path)
      command:
        - /bin/sh
      args:
        - '-c'
        - git clone -v $(params.repo) ./source
---
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: list
spec:
  workspaces:
    - name: source
  steps:
    - name: list
      image: alpine
      workingDir: $(workspaces.source.path)
      command:
        - /bin/sh
      args:
        - "-c"
        - ls ./source
EOF
```

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: clone-and-list
spec:
  workspaces:
    - name: codebase
  tasks:
    - name: clone
      taskRef:
        name: clone
      workspaces:
        - name: source
          workspace: codebase
    - name: list
      taskRef:
        name: list
      workspaces:
        - name: source
          workspace: codebase
      runAfter:
        - clone
EOF
```

<br/>

```
$ tkn pipeline start clone-and-list --showlog
```

<br/>

```
? Name for the workspace : codebase
? Value of the Sub Path :
? Type of the Workspace : emptyDir
? Type of EmptyDir :
```

<br/>

**result:**

<br/>

```
[list : list] ls: ./source: No such file or directory

failed to get logs for task list : container step-list has failed  : [{"key":"StartedAt","value":"2021-10-02T21:09:56.952Z","type":3}]
```

<br/>

To share the content across tasks, you will need to use a persistent volume, which you will do in the next section.

<br/>

### Persisting data within a pipeline

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: tekton-pv
spec:
  storageClassName: manual
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteMany
    - ReadWriteOnce
    - ReadOnlyMany
  hostPath:
    path: "/mnt/data"
EOF
```

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: tekton-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
EOF
```

<br/>

```
$ tkn pipeline start clone-and-list \
  -w name=codebase,claimName=tekton-pvc \
  --showlog
```

Если выполнить второй раз - будет ошибка, т.к. данные из pvc не были удалены.

<br/>

### Cleaning up with finally

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: cleanup
spec:
  workspaces:
    - name: source
  steps:
    - name: remove-source
      image: registry.access.redhat.com/ubi8/ubi
      command:
        - /bin/bash
      args:
        - "-c"
        - "rm -rf $(workspaces.source.path)/source"
    - name: message
      image: registry.access.redhat.com/ubi8/ubi
      command:
        - /bin/bash
      args:
        - "-c"
        - echo All files were deleted
EOF
```

<br/>

```yaml
$ cat << 'EOF' | kubectl apply -f -
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: clone-and-list
spec:
  workspaces:
    - name: codebase
  tasks:
    - name: clone
      taskRef:
        name: clone
      workspaces:
        - name: source
          workspace: codebase
    - name: list
      taskRef:
        name: list
      workspaces:
        - name: source
          workspace: codebase
      runAfter:
        - clone
  finally:
    - name: clean
      taskRef:
        name: cleanup
      workspaces:
        - name: source
          workspace: codebase
EOF
```

<br/>

```
$ tkn pipeline start clone-and-list \
  -w name=codebase,claimName=tekton-pvc \
  --showlog
```

<br/>

### Using workspaces in pipeline runs

<br/>

```
$ vi ~/tmp/pipelinerun.yaml
```

<br/>

```yaml
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
    generateName: clone-and-ls-pr-
spec:
    pipelineRef:
        name: clone-and-list
    workspaces:
        - name: codebase
          persistentVolumeClaim:
              claimName: tekton-pvc
```

<br/>

```
$ kubectl create -f ~/tmp/pipelinerun.yaml
```

<br/>

```
$ tkn pr logs clone-and-ls-pr-hsqfz -f
```

<br/>

### Using volume claim templates

Instead of specifying a persistent volume claim directly, you can also ask Tekton to create a temporary one for you. This can be useful when you don't need to persist data outside of your pipelines.

<br/>

```
$ vi ~/tmp/pvc-template.yaml
```

<br/>

```yaml
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
    generateName: clone-and-ls-pr-
spec:
    pipelineSpec:
        workspaces:
            - name: codebase
        tasks:
            - name: clone
              taskRef:
                  name: clone
              workspaces:
                  - name: source
                    workspace: codebase
            - name: list
              taskRef:
                  name: list
              workspaces:
                  - name: source
                    workspace: codebase
              runAfter:
                  - clone
    workspaces:
        - name: codebase
          volumeClaimTemplate:
              spec:
                  accessModes:
                      - ReadWriteOnce
                  resources:
                      requests:
                          storage: 1Gi
```

<br/>

```
$ kubectl create -f ~/tmp/pvc-template.yaml
```

<br/>

```
$ tkn pr logs -f clone-and-ls-pr-xwsnd
```

<br/>

This volume claim template creates a new PVC for each pipeline execution.
