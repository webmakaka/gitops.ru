---
layout: page
title: FluxCD v2
description: FluxCD v2
keywords: linux, kubernetes, FluxCD
permalink: /study/videos/containers/kubernetes/ci-cd/fluxcd/
---

# FluxCD v2

<br/>

### [FluxCD v2 With GitOps Toolkit - Kubernetes Deployment And Sync Mechanism](/study/videos/containers/kubernetes/ci-cd/fluxcd/fluxcd-v2-with-gitops-toolkit/)

### [Supercharge your Kubernetes deployments with Flux v2 and GitHub](/study/videos/containers/kubernetes/ci-cd/fluxcd/supercharge-your-kubernetes-deployments-with-flux-v2-and-github/)

<br/>

### GitOps Days 2021 Handling Dependencies with Flux - Jason Morgan

https://www.youtube.com/watch?v=laMwuG8r7Tw

<br/>

[Инсталляция Linkerd](/containers/kubernetes/tools/service-mesh/linkerd/setup/)

<br/>

```
$ cd ~/tmp
$ git clone https://github.com/BuoyantIO/gitops_examples
```

<br/>

```
$ kubectl apply -f gitops_examples/flux/runtime/manifests/runtime_git.yaml
```

<br/>

```
$ kubectl get gitrepositories -A
NAMESPACE     NAME     URL                                                READY   STATUS                                                            AGE
flux-system   gitops   https://github.com/BuoyantIO/gitops_examples.git   True    Fetched revision: main/6bcab49784782321cfd688592822b10f6673365f   4m25s
```

<br/>

```
$ kubectl apply -f gitops_examples/flux/runtime/manifests/dev_cluster.yaml
```

<br/>

```
$ watch kubectl get kustomizations.kustomize.toolkit.fluxcd.io -A
```

<br/>

```
$ linkerd check
$ linkerd viz check
```

<br/>

```
$ kubectl apply -f gitops_examples/flux/apps/manifests/podinfo.yaml
```

<br/>

```
$ linkerd viz dashboard
```

<br/>

```
$ kubectl port-forward svc/frontend 8080:8080
```

<br/>

```
$ vi gitops_examples/flux/apps/source/podinfo/patch.yaml
```

<br/>

меняем цвет

<br/>

git add / git commit / git push

<br/>

```
$ flux reconcile kustomization podinfo
```

<br/>

```
$ linkerd viz dashboard
```

<br/>

Обратить внимание на Traffic Splits

<br/>

### Cloud Native Applications Gitops with Flux and Flagger

https://www.youtube.com/watch?v=kbAKUKaMA1w

https://github.com/Tiggel/flux-app-demo/

<br/>

```
// Создается приватный репо flux-fleet
$ flux bootstrap github \
    --owner $GITHUB_USER \
    --repository flux-app-demo \
    --branch canary \
    --path "./config/play/flux" \
    --personal
```

<br/>

Меняем версию podinfo на 5.2.0 в branch canary

https://github.com/Tiggel/flux-app-demo/commit/69969f5818c3da7a14e4493fe0a74b086b753929

<br/>

https://docs.flagger.app/tutorials/istio-progressive-delivery

<br/>

```
$ flux reconcile ks canary --with-source
```

<br/>

```
$ watch curl http://podinfo-canary:9898/status/500
```
