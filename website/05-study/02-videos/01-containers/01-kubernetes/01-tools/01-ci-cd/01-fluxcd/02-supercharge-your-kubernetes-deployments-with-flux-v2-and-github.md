---
layout: page
title: Supercharge your Kubernetes deployments with Flux v2 and GitHub
description: Supercharge your Kubernetes deployments with Flux v2 and GitHub
keywords: linux, kubernetes, FluxCD, Kustomize
permalink: /study/videos/containers/kubernetes/tools/ci-cd/fluxcd/supercharge-your-kubernetes-deployments-with-flux-v2-and-github/
---

# Supercharge your Kubernetes deployments with Flux v2 and GitHub

<br/>

https://www.youtube.com/watch?v=N6UCKF7JD7k&list=PLG9qZAczREKmCq6on_LG8D0uiHMx1h3yn&index=1

<br/>

## 01-Supercharge your Kubernetes deployments with Flux v2 and GitHub - Introduction

<br/>

### Подключение к бесплатному облаку от Google

https://shell.cloud.google.com/

<br/>

**Инсталлим google-cloud-sdk**

https://cloud.google.com/sdk/docs/install

<br/>

```
$ gcloud auth login
$ gcloud cloud-shell ssh
```

<br/>

1. Инсталляция [MiniKube](/containers/kubernetes/setup/minikube/)

**Испольновалась версия KUBERNETES_VERSION=v1.22.2**

2. Инсталляция [Kubectl](/containers/kubernetes/tools/kubectl/)

3. Инсталляция fluxcd

<br/>

```
$ curl -s https://fluxcd.io/install.sh | sudo bash
```

<br/>

```
$ flux --version
flux version 0.21.1
```

<br/>

```
$ flux check
► checking prerequisites
✔ Kubernetes 1.22.2 >=1.19.0-0
► checking controllers
✔ helm-controller: deployment ready
► ghcr.io/fluxcd/helm-controller:v0.12.1
✔ kustomize-controller: deployment ready
► ghcr.io/fluxcd/kustomize-controller:v0.16.0
✔ notification-controller: deployment ready
► ghcr.io/fluxcd/notification-controller:v0.18.1
✔ source-controller: deployment ready
► ghcr.io/fluxcd/source-controller:v0.17.2
✔ all checks passed
```

<br/>

4. Инсталляция gh Linux

<br/>

```
$ cd ~/tmp
```

<br/>

```
$ vi gh.sh
```

<br/>

```
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo gpg --dearmor -o /usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable master" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update
sudo apt install gh
```

<br/>

```
$ chmod +x gh.sh
$ ./gh.sh
```

<br/>

```
// Чтобы создавался origin на ssh а не https
$ gh config set git_protocol ssh -h github.com
```

<br/>

```
$ git config --global user.name "<GITHUB_USERNAME>"
$ git config --global user.email "<GITHUB_EMAIL>"
```

<br/>

// Страница генерации тогена  
https://github.com/settings/tokens

<br/>

```
$ export INGRESS_HOST=$(minikube --profile ${PROFILE} ip)

$ echo ${INGRESS_HOST}

$ export GITHUB_USER=<YOUR_GITHUB_USERNAME>

$ export GITHUB_TOKEN=<YOUR_GITHUB_TOKEN>
```

<br/>

```
// Будет создано приватное репо flux-infra
$ flux bootstrap github \
    --owner=${GITHUB_USER} \
    --repository=flux-infra \
    --branch=master \
    --path=app-cluster \
    --personal
```

<br/>

# 02-Kubernetes deployments with Flux v2 introduction to kustomize

<br/>

```
$ curl -s "https://raw.githubusercontent.com/\
kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash && chmod +x kustomize && sudo mv kustomize /usr/local/bin/
```

<br/>

```
$ mkdir -p ~/tmp && cd ~/tmp

$ git clone https://github.com/webmak1/realtimeapp-infra

$ cd ~/tmp/realtimeapp-infra/deploy/overlays/dev
$ kustomize build

// $ kustomize build | kubeclt apply -f -
```

<br/>

# 03-Kubernetes deployments with Flux v2 Deploying Manifests

```
$ mkdir -p ~/project/dev && cd ~/project/dev
$ git clone https://github.com/${GITHUB_USER}/flux-infra
$  cd flux-infra
```

<br/>

```
$ flux create source git realtimeapp-infra \
    --url https://github.com/$GITHUB_USER/realtimeapp-infra \
    --branch master \
    --interval 30s \
    --export > ./app-cluster/realtimeapp-source.yaml
```

<br/>

```
$ cat ./app-cluster/realtimeapp-source.yaml
```

<br/>

```
$ flux create kustomization realtimeapp-dev \
    --source realtimeapp-infra \
    --path "./deploy/overlays/dev" \
    --prune true \
    --validation client \
    --interval 1m \
    --health-check="Deployment/realtime-dev.realtime-dev" \
    --health-check="Deployment/redis-dev.realtime-dev" \
    --health-check-timeout=2m \
    --export > ./app-cluster/realtimeapp-dev.yaml
```

<br/>

```
$ flux create kustomization realtimeapp-prd \
    --source realtimeapp-infra \
    --path "./deploy/overlays/prd" \
    --prune true \
    --validation client \
    --interval 1m \
    --health-check="Deployment/realtime-prd.realtime-prd" \
    --health-check="Deployment/redis-prd.realtime-prd" \
    --health-check-timeout=2m \
    --export > ./app-cluster/realtimeapp-prd.yaml
```

<br/>

```
$ cat ./app-cluster/realtimeapp-source.yaml
```

<br/>

```
$ git add --all

$ git commit -m "Added source and kustomization"

$ git push
```

<br/>

```
$ watch flux get kustomizations
```

<br/>

```
$ kubectl get ns
NAME              STATUS   AGE
default           Active   6d9h
flux-system       Active   18m
kube-node-lease   Active   6d9h
kube-public       Active   6d9h
kube-system       Active   6d9h
realtime-dev      Active   82s
realtime-prd      Active   83s
```

<br/>

```
$ kubectl get pods -n realtime-dev
NAME                          READY   STATUS    RESTARTS   AGE
realtime-dev-869b5674-h6gd5   1/1     Running   0          115s
redis-dev-589977c5c6-7pccx    1/1     Running   0          115s
```

<br/>

```
$ flux reconcile kustomization realtimeapp-dev
```

<br/>

https://github.com/webmak1/realtimeapp

New Release

1.0.2

Publish

В Actions должен пойти билд.

Там срабатывает kustomize edit который меняет версию.

<br/>

# 04-Kubernetes deployments with Flux v2 Monitoring and Alerting

(Пропустил этот шаг)

<br/>

```
$ cd ~/project/dev/flux-infra
```

<br/>

```
$ flux create source git monitoring \
    --url https://github.com/fluxcd/flux2 \
    --branch master \
    --interval 30m \
    --export > ./app-cluster/monitor-source.yaml
```

<br/>

```
$ flux create kustomization monitoring \
    --source monitoring \
    --path "./manifests/monitoring" \
    --prune true \
    --interval 1h \
    --health-check="Deployment/prometheus.flux-system" \
    --health-check="Deployment/grafana.flux-system" \
    --export > ./app-cluster/monitor-kustomization.yaml
```

<br/>

```
$ watch flux get kustomizations
```

<br/>

```
$ kubectl -n flux-system port-forward svc/grafana 3000:3000
```

<br/>

Далее Alerting, какие-то teams и т.д. Наверное, имеет смысл пересмотреть.

<br/>

```
$ flux get alert-providers
```

<br/>

# 05-Kubernetes Deployments with Flux v2 Helm Basics

<br/>

```
$ flux create source helm bitnami \
    --url https://charts.bitnami.com/bitnami \
    --interval 1m0s \
    --export > ./app-cluster/helmrepo-bitnami.yaml
```

<br/>

```
$ flux create helmrelease redis \
    --source=HelmRepository/bitnami \
    --chart redis \
    --release-name redis \
    --target-namespace default \
    --interval 5m0s \
    --export > ./app-cluster/helmrelease-redis.yaml
```

<br/>

```
$ flux get sources helm
```

<br/>

```
$ export HELM_EXPERIMENTAL_OCI=1
$ helm chart save . realtimeapp:1.0.2

// Отправляем в helm registry
$ helm chart save . gebareg.azurecr.io/realtimeapp:1.0.2
$ helm registry login gebareg.azurecr.io
$ helm chart push gebareg.azurecr.io/realtimeapp:1.0.2

// Защищенный паролем репо
$ kubectl create secret generic acr --from-literal username=gebareg --from-literal "password=THEPASSWORD" -n flux-system
```

<br/>

```
$ flux create source helm realtimeapp \
    --url https://gebareg.auzrecr.io/helm/v1/repo/ \
    --interval 1m0s \
    --secret-ref acr \
    --export > ./app-cluster/helmrepo-realtimeapp.yaml
```

<br/>

```
$ flux create helmrelease realtimeapp \
    --source=HelmRepository/realtimeapp \
    --chart realtimeapp \
    --release-name realtimeapp \
    --target-namespace default \
    --interval 5m0s \
    --export > ./app-cluster/helmrelease-realtimeapp.yaml
```

Добавили некоторые изменения в конфиги редиса и realtimeapp.
