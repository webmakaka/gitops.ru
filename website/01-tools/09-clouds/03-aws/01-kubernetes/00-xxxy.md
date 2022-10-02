---
layout: page
title: Clouds Amazon (AWS) - Kubernetes Setup
description: Clouds Amazon (AWS) - Kubernetes Setup
keywords: Clouds Amazon (AWS) - Kubernetes Setup
permalink: /tools/clouds/aws/kubernetes/setup/manual/xxx/
---

<br/>

```
//

// install nodejs


//
$ npm install -g aws-cdk

$ cdk --version
2.43.1 (build c1ebb85)
```

<br/>

```
$ kubectl version --client
```

<!-- <br/>

### cdk

```
$ mkdir cdk-workshop && cd cdk-workshop

$ cdk init sample-app --language typescript
$ cd cdk-workshop
$ npm run watch
``` -->

```
$ mkdir ~/tmp/aws
$ cd ~/tmp/aws
$ git clone https://github.com/yjw113080/aws-cdk-eks-multi-region-skeleton
$ cd aws-cdk-eks-multi-region-skeleton/

$ ncu -u
$ npm install
$ npm run watch

```
