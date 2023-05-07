---
layout: page
title: Hardening Default GKE Cluster Configurations
description: Hardening Default GKE Cluster Configurations
keywords: Hardening Default GKE Cluster Configurations
permalink: /devops/clouds/google/gke/qwiklabs/kubernetes-best-practices-security/hardening-default-gke-cluster-configurations/
---

# [GSP476] Hardening Default GKE Cluster Configurations

<br/>

Делаю:  
09.06.2019

https://www.qwiklabs.com/focuses/5158?parent=catalog

<br>

### Lab setup

    $ export MY_ZONE=us-central1-a
    $ gcloud container clusters create simplecluster --zone $MY_ZONE --num-nodes 2

<br/>

### Run a Google Cloud-SDK pod

    $ kubectl run -it --rm gcloud --image=google/cloud-sdk:latest --restart=Never -- bash

    # curl -s -H "Metadata-Flavor: Google" http://metadata.google.internal/computeMetadata/v1/instance/name

    # curl -s -H "Metadata-Flavor: Google" http://metadata.google.internal/computeMetadata/v1/instance/attributes/

    # curl -s -H "Metadata-Flavor: Google" http://metadata.google.internal/computeMetadata/v1/instance/attributes/kube-env

<br/>

В общем доступ ко всему есть

<br/>

Therefore, in any of the following situations:

A flaw that allows for SSRF in a pod application
An application or library flaw that allow for RCE in a pod
An internal user with the ability to create or exed into a pod

<br/>

### Leverage the Permissions Assigned to this Node Pool's Service Account

    # curl -s -H "Metadata-Flavor: Google" http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/scopes

    # exit

<!--
<br/>

### Deploy a pod that mounts the host filesystem

```
$ cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: hostpath
spec:
  containers:
  - name: hostpath
    image: google/cloud-sdk:latest
    command: ["/bin/bash"]
    args: ["-c", "tail -f /dev/null"]
    volumeMounts:
    - mountPath: /rootfs
      name: rootfs
  volumes:
  - name: rootfs
    hostPath:
      path: /
EOF
```

<br/>

    $ kubectl get pod

    $ kubectl exec -it hostpath -- bash

    $ chroot /rootfs /bin/bash

    $ kubectl delete pod hostpath
 -->

<br/>

### Understand the available controls

The next steps of this demo will cover:

Disabling the Legacy GCE Metadata API Endpoint - By specifying a custom metadata key and value, the v1beta1 metadata endpoint will no longer be available from the instance.

Enable Metadata Concealment - Passing an additional configuration during cluster and/or node pool creation, a lightweight proxy will be installed on each node that proxies all requests to the Metadata API and prevents access to sensitive endpoints.

Enable and configure PodSecurityPolicy - Configuring this option on a GKE cluster will add the PodSecurityPolicy Admission Controller which can be used to restrict the use of insecure settings during Pod creation. In this demo's case, preventing containers from running as the root user and having the ability to mount the underlying host filesystem.

<br/>

### Deploy a second node pool

    $ gcloud beta container node-pools create second-pool --cluster=simplecluster --zone=$MY_ZONE --num-nodes=1 --metadata=disable-legacy-endpoints=true --workload-metadata-from-node=SECURE

<br/>

### Run a Google Cloud-SDK pod

Теперь все, вроде как, должно запускаться в chroot

    $ kubectl run -it --rm gcloud --image=google/cloud-sdk:latest --restart=Never --overrides='{ "apiVersion": "v1", "spec": { "securityContext": { "runAsUser": 65534, "fsGroup": 65534 }, "nodeSelector": { "cloud.google.com/gke-nodepool": "second-pool" } } }' -- bash

<br/>

    $ curl -s http://metadata.google.internal/computeMetadata/v1beta1/instance/name
    В доступе отказано

    $ curl -s -H "Metadata-Flavor: Google" http://metadata.google.internal/computeMetadata/v1/instance/attributes/kube-env
    В доступе отказано

    $ curl -s -H "Metadata-Flavor: Google" http://metadata.google.internal/computeMetadata/v1/instance/name
    Доступ получен

    $ exit

<br/>

### Deploy PodSecurityPolicy objects

    $ kubectl create clusterrolebinding clusteradmin --clusterrole=cluster-admin --user="$(gcloud config list account --format 'value(core.account)')"

```
$ cat <<EOF | kubectl apply -f -
---
apiVersion: extensions/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restrictive-psp
  annotations:
    seccomp.security.alpha.kubernetes.io/allowedProfileNames: 'docker/default'
    apparmor.security.beta.kubernetes.io/allowedProfileNames: 'runtime/default'
    seccomp.security.alpha.kubernetes.io/defaultProfileName:  'docker/default'
    apparmor.security.beta.kubernetes.io/defaultProfileName:  'runtime/default'
spec:
  privileged: false
  # Required to prevent escalations to root.
  allowPrivilegeEscalation: false
  # This is redundant with non-root + disallow privilege escalation,
  # but we can provide it for defense in depth.
  requiredDropCapabilities:
    - ALL
  # Allow core volume types.
  volumes:
    - 'configMap'
    - 'emptyDir'
    - 'projected'
    - 'secret'
    - 'downwardAPI'
    # Assume that persistentVolumes set up by the cluster admin are safe to use.
    - 'persistentVolumeClaim'
  hostNetwork: false
  hostIPC: false
  hostPID: false
  runAsUser:
    # Require the container to run without root privileges.
    rule: 'MustRunAsNonRoot'
  seLinux:
    # This policy assumes the nodes are using AppArmor rather than SELinux.
    rule: 'RunAsAny'
  supplementalGroups:
    rule: 'MustRunAs'
    ranges:
      # Forbid adding the root group.
      - min: 1
        max: 65535
  fsGroup:
    rule: 'MustRunAs'
    ranges:
      # Forbid adding the root group.
      - min: 1
        max: 65535
EOF
```

<br/>

```
$ cat <<EOF | kubectl apply -f -
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: restrictive-psp
rules:
- apiGroups:
  - extensions
  resources:
  - podsecuritypolicies
  resourceNames:
  - restrictive-psp
  verbs:
  - use
EOF

```

<br/>

```
$ cat <<EOF | kubectl apply -f -
---
# All service accounts in kube-system
# can 'use' the 'permissive-psp' PSP
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: restrictive-psp
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: restrictive-psp
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:authenticated
EOF

```

Note: In a real environment, consider replacing the system:authenticated user in the RoleBinding with the specific user or service accounts that you want to have the ability to create pods in the default namespace.

<br/>

### Enable PodSecurity policy

    $ gcloud beta container clusters update simplecluster --zone $MY_ZONE --enable-pod-security-policy

<br/>

### Deploy a blocked pod that mounts the host filesystem

    $ gcloud iam service-accounts create demo-developer

    $ MYPROJECT=$(gcloud config list --format 'value(core.project)')

    $ gcloud projects add-iam-policy-binding "${MYPROJECT}" --role=roles/container.developer --member="serviceAccount:demo-developer@${MYPROJECT}.iam.gserviceaccount.com"

    $ gcloud iam service-accounts keys create key.json --iam-account "demo-developer@${MYPROJECT}.iam.gserviceaccount.com"

    $ gcloud auth activate-service-account --key-file=key.json

    $ gcloud container clusters get-credentials simplecluster --zone $MY_ZONE

<br/>

```
$ cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: hostpath
spec:
  containers:
  - name: hostpath
    image: google/cloud-sdk:latest
    command: ["/bin/bash"]
    args: ["-c", "tail -f /dev/null"]
    volumeMounts:
    - mountPath: /rootfs
      name: rootfs
  volumes:
  - name: rootfs
    hostPath:
      path: /
EOF
```

// Так и должно быть
Error from server (Forbidden): error when creating "STDIN": pods "hostpath" is forbidden: unable to validate against any pod security policy: [spec.volumes[0]: Invalid value: "hostPath": hostPath volumes are not allowed to be used

<br/>

```
$ cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: hostpath
spec:
  securityContext:
    runAsUser: 1000
    fsGroup: 2000
  containers:
  - name: hostpath
    image: google/cloud-sdk:latest
    command: ["/bin/bash"]
    args: ["-c", "tail -f /dev/null"]
EOF
```

<br/>

    $ kubectl get pod hostpath -o=jsonpath="{ .metadata.annotations.kubernetes\.io/psp }"
