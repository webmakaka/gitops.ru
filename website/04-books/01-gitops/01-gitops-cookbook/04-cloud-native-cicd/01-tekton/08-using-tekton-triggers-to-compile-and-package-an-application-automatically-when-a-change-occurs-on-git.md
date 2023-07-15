---
layout: page
title: GitOps Cookbook - Cloud Native CI/CD - Tekton - Using Tekton Triggers to Compile and Package an Application Automatically When a Change Occurs on Git
description: GitOps Cookbook - Cloud Native CI/CD - Tekton - Using Tekton Triggers to Compile and Package an Application Automatically When a Change Occurs on Git
keywords: GitOps Cookbook - Cloud Native CI/CD, Tekton, Using Tekton Triggers to Compile and Package an Application Automatically When a Change Occurs on Git
permalink: /books/gitops/gitops-cookbook/cloud-native-cicd/tekton/using-tekton-triggers-to-compile-and-package-an-application-automatically-when-a-change-occurs-on-git/
---

<br/>

# [Book] [FAIL!] GitOps Cookbook: 06. Cloud Native CI/CD: Tekton: 6.8 Using Tekton Triggers to Compile and Package an Application Automatically When a Change Occurs on Git

<br/>

**Делаю:**  
16.06.2023

<br/>

This will create a new ServiceAccount named tekton-triggers-sa that has the permissions needed to interact with the Tekton Pipelines component.

<br/>

```yaml
$ cat << 'EOF' | kubectl create -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tekton-triggers-sa
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: triggers-example-eventlistener-binding
subjects:
- kind: ServiceAccount
  name: tekton-triggers-sa
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: tekton-triggers-eventlistener-roles
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: triggers-example-eventlistener-clusterbinding
subjects:
- kind: ServiceAccount
  name: tekton-triggers-sa
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: tekton-triggers-eventlistener-clusterroles
EOF
```

<br/>

```yaml
$ cat << 'EOF' | kubectl create -f -
apiVersion: triggers.tekton.dev/v1alpha1
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
        name: tekton-greeter-pipeline-webhook-$(uid)
      spec:
        params:
          - name: GIT_REPO
            value: $(tt.params.git-repo-url)
          - name: GIT_REF
            value: $(tt.params.git-revision)
        serviceAccountName: tekton-triggers-example-sa
        pipelineRef:
          name: tekton-greeter-pipeline-hub
        workspaces:
        - name: app-source
          persistentVolumeClaim:
            claimName: app-source-pvc
        - name: maven-settings
          emptyDir: {}
---
apiVersion: triggers.tekton.dev/v1alpha1
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
apiVersion: triggers.tekton.dev/v1alpha1
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

```
$ kubectl get pods
$ kubectl get svc
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
// Ничего не произошло
$ tkn pipelinerun ls
```

<br/>

```
$ kubectl get pods
$ kubectl logs el-tekton-greeter-eventlistener-79f47f896-kvjx7 | jq
```

<br/>

```json
{
  "severity": "error",
  "timestamp": "2023-06-16T17:01:16.960Z",
  "logger": "eventlistener",
  "caller": "sink/sink.go:606",
  "message": "problem creating obj: &errors.errorString{s:\"couldn't create resource with group version kind \\\"tekton.dev/v1beta1, Resource=pipelineruns\\\": admission webhook \\\"validation.webhook.pipeline.tekton.dev\\\" denied the request: validation failed: Invalid resource name: length must be no more than 63 characters: metadata.name\"}",
  "commit": "2ec8bc6-dirty",
  "eventlistener": "tekton-greeter-eventlistener",
  "namespace": "default",
  "/triggers-eventid": "30b02245-1943-4db5-96fb-a5e9aa50f1a2",
  "eventlistenerUID": "210d2e53-d96d-4096-b2d7-4af7239d86b3",
  "/trigger": ""
}
{
  "severity": "error",
  "timestamp": "2023-06-16T17:01:16.960Z",
  "logger": "eventlistener",
  "caller": "sink/sink.go:446",
  "message": "couldn't create resource with group version kind \"tekton.dev/v1beta1, Resource=pipelineruns\": admission webhook \"validation.webhook.pipeline.tekton.dev\" denied the request: validation failed: Invalid resource name: length must be no more than 63 characters: metadata.name",
  "commit": "2ec8bc6-dirty",
  "eventlistener": "tekton-greeter-eventlistener",
  "namespace": "default",
  "/triggers-eventid": "30b02245-1943-4db5-96fb-a5e9aa50f1a2",
  "eventlistenerUID": "210d2e53-d96d-4096-b2d7-4af7239d86b3",
  "/trigger": ""
}
```
