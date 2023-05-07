---
layout: page
title: Kubernetes Custom Resource Definitions
description: Kubernetes Custom Resource Definitions
keywords: devops, linux, kubernetes, Custom Resource Definitions, crds
permalink: /devops/containers/kubernetes/basics/custom-resource-definitions/
---

# Kubernetes Custom Resource Definitions (CRDs)

<br/>

Делаю:
22.04.2020

<br/>

https://github.com/burrsutter/9stepsawesome/blob/master/11_crds.adoc

<br/>

https://github.com/burrsutter/9stepsawesome/tree/master/pizzas

<br/>

```
$ cat <<EOF | kubectl apply -f -
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: pizzas.mykubernetes.burrsutter.com
  labels:
    app: pizzamaker
    mylabel: stuff
spec:
  group: mykubernetes.burrsutter.com
  scope: Namespaced
  version: v1beta2
  names:
    kind: Pizza
    listKind: PizzaList
    plural: pizzas
    singular: pizza
    shortNames:
    - pz
  validation:
    openAPIV3Schema:
      type: object
      properties:
        spec:
          type: object
          properties:
            toppings:
              type: array
            sauce:
              type: string
EOF
```

<br/>

    $ kubectl get crds
    NAME                                 CREATED AT
    pizzas.mykubernetes.burrsutter.com   2020-04-21T20:50:35Z

<br/>

    $ kubectl create namespace pizzahat
    $ kubectl config set-context --current --namespace=pizzahat

<br/>

```
$ cat <<EOF | kubectl apply -f -
apiVersion: mykubernetes.burrsutter.com/v1beta2
kind: Pizza
metadata:
  name: burrcheese
spec:
  toppings:
  - mozzarella
  sauce: regular
EOF
```

<br/>

    $ kubectl get pizzas
    NAME         AGE
    burrcheese   47s


    $ kubectl describe pizza burrcheese

    $ kubectl api-resources | grep burr
    pizzas                            pz           mykubernetes.burrsutter.com    true         Pizza

<br/>

**Make more Pizzas**

```
$ cat <<EOF | kubectl apply -f -
apiVersion: mykubernetes.burrsutter.com/v1beta2
kind: Pizza
metadata:
  name: burrmeats
spec:
  toppings:
  - mozzarella
  - pepperoni
  - sausage
  - bacon
  sauce: extra
EOF
```

```
$ cat <<EOF | kubectl apply -f -
apiVersion: mykubernetes.burrsutter.com/v1beta2
kind: Pizza
metadata:
  name: burrveggie2
spec:
  toppings:
  - mozzarella
  - black olives
  sauce: extra
EOF
```

    $ kubectl get pizzas --all-namespaces

    $ kubectl get pizzas --all-namespaces
    NAMESPACE   NAME          AGE
    pizzahat    burrcheese    5m16s
    pizzahat    burrmeats     5s
    pizzahat    burrveggie2   10s


    $ kubectl delete pizzas --all

<br/>

### Controllers

    $ kubectl create namespace metacontroller

    $ kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/metacontroller/master/manifests/metacontroller-rbac.yaml

    $ kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/metacontroller/master/manifests/metacontroller.yaml

    $ kubectl config set-context --current --namespace=metacontroller

    $ kubectl get pods
    NAME               READY   STATUS    RESTARTS   AGE
    metacontroller-0   1/1     Running   0          2m26s

```
$ cat <<EOF | kubectl apply -f -
apiVersion: metacontroller.k8s.io/v1alpha1
kind: CompositeController
metadata:
  name: pizza-controller
spec:
  generateSelector: true
  parentResource:
    apiVersion: mykubernetes.burrsutter.com/v1beta2
    resource: pizzas
  childResources:
  - apiVersion: v1
    resource: pods
    updateStrategy:
      method: Recreate
  hooks:
    sync:
      webhook:
        url: http://pizza-controller.pizzahat:8080/sync
EOF
```

    $ touch ~/tmp/sync.py

```
$ cat > ~/tmp/sync.py << EOF
from BaseHTTPServer import BaseHTTPRequestHandler, HTTPServer
import json
import logging

class Controller(BaseHTTPRequestHandler):
  def sync(self, parent, children):

    # Compute status based on observed state.
    desired_status = {
       "pods": 1
    }

    # Collect the specifications
    name = parent["metadata"]["name"]
    sauce = parent.get("spec", {}).get("sauce")
    toppings = parent.get("spec",{}).get("toppings")
    print("\n\nsauce: %s" % sauce)
    print("toppings: %s" % toppings)
    for topping in toppings:
      print(topping)

    stuff = ' ' + sauce + ' ' + ' '.join(toppings) + ' of ' + name

    print stuff

    # Generate the desired child object(s)

    desired_pods = [
      {
        "apiVersion": "v1",
        "kind": "Pod",
        "metadata": {
          "name": name
        },
        "spec": {
          "restartPolicy": "OnFailure",
          "containers": [
            {
              "name": "pizza",
              "image": "busybox",
              "command": ["echo", "requested pizza: %s" % stuff]
            }
          ]
        }
      }
    ]

    return {"status": desired_status, "children": desired_pods}

  def do_POST(self):
    # Serve the sync() function as a JSON webhook.

    observed = json.loads(self.rfile.read(int(self.headers.getheader("content-length"))))
    desired = self.sync(observed["parent"], observed["children"])

    self.send_response(200)
    self.send_header("Content-type", "application/json")
    self.end_headers()
    self.wfile.write(json.dumps(desired))

HTTPServer(("", 8080), Controller).serve_forever()
EOF
```

    $ kubectl -n pizzahat create configmap pizza-controller --from-file=/home/marley/tmp/sync.py

    $ kubectl -n pizzahat apply -f webhook-py.yaml

```
$ cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pizza-controller
spec:
  replicas: 1
  selector:
    matchLabels:
      app: pizza-controller
  template:
    metadata:
      labels:
        app: pizza-controller
    spec:
      containers:
      - name: controller
        image: python:2.7
        command: ["python", "/hooks/sync.py"]
        volumeMounts:
        - name: hooks
          mountPath: /hooks
      volumes:
      - name: hooks
        configMap:
          name: pizza-controller
---
apiVersion: v1
kind: Service
metadata:
  name: pizza-controller
spec:
  selector:
    app: pizza-controller
  ports:
  - port: 8080
EOF
```

**Deploy some Pizzas**

```
$ cat <<EOF | kubectl -n pizzahat  apply -f -
apiVersion: mykubernetes.burrsutter.com/v1beta2
kind: Pizza
metadata:
  name: burrcheese
spec:
  toppings:
  - mozzarella
  sauce: regular
EOF
```

```
$ cat <<EOF | kubectl -n pizzahat  apply -f -
apiVersion: mykubernetes.burrsutter.com/v1beta2
kind: Pizza
metadata:
  name: burrmeats
spec:
  toppings:
  - mozzarella
  - pepperoni
  - sausage
  - bacon
  sauce: extra
EOF
```

```
$ cat <<EOF | kubectl -n pizzahat  apply -f -
apiVersion: mykubernetes.burrsutter.com/v1beta2
kind: Pizza
metadata:
  name: burrveggie2
spec:
  toppings:
  - mozzarella
  - black olives
  sauce: extra
EOF
```

Ничего не заработало! pizza-controller не запустился!

    $ kubectl logs burrveggie -n pizzahat
    $ kubectl delete pizza burrveggie
    $ kubectl delete namespace pizzahat

<br/>

### Kafka via OperatorHub

http://operatorhub.io/

    $ kubectl create namespace franz

    $ kubectl config set-context --current --namespace=franz

    $ curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/0.12.0/install.sh | bash -s 0.12.0

    $ kubectl create -f https://operatorhub.io/install/strimzi-kafka-operator.yaml

    $ kubectl get csv -n operators

    $ kubectl get crds | grep kafka

    $ watch kubectl get pods

<br/>

    $ kubectl apply -f https://raw.githubusercontent.com/burrsutter/9stepsawesome/master/kubefiles/kafka-strimzi-minikube.yml

<br/>

    $ kubectl get kafkas
    NAME           DESIRED KAFKA REPLICAS   DESIRED ZK REPLICAS
    burr-cluster   3                        3

<br/>

    $ kubectl get pods
    NAME                                           READY   STATUS    RESTARTS   AGE
    burr-cluster-entity-operator-84b7bfc5c-8vjc4   2/2     Running   0          3m10s
    burr-cluster-kafka-0                           2/2     Running   0          3m43s
    burr-cluster-kafka-1                           2/2     Running   0          3m43s
    burr-cluster-kafka-2                           2/2     Running   0          3m43s
    burr-cluster-zookeeper-0                       2/2     Running   0          4m38s
    burr-cluster-zookeeper-1                       2/2     Running   0          4m38s
    burr-cluster-zookeeper-2                       2/2     Running   0          4m38s

<br/>

**Clean up**

    $ kubectl delete kafka burr-cluster
    $ kubectl delete namespace franz
