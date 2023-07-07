---
title: 'Most Wanted Kubernetes Objects Explained - Episode 01'
subtitle: Pods, Replication Controllers, Replica Sets, Deployments
description: "Kubernetes is now becoming a new kid in town because its easy to deploy, scale, and manage thousands of containerized applications more quicker. Let's start from scratch."
image: /img/post-imgs/k8s-objects-01/kubernetes-objects-01_kubernetes.jpg
categories: ["DevOps"]
type: "regular" # available types: [featured/regular]
draft: false
date: 2021-03-19
---

Kubernetes is now becoming a new kid in town. Now, Kubernetes is in action, because its easy to deploy scale and manage thousands of containerized applications more quicker. Now, most of cloud providers such as AWS, Azure , and Google privde managed kubernetes services run our deployments. This article would helps anyone who steping into the kubernetes world. This will also would helpful for someone who getting ready for Certified Kubernetes Administrator exam. Now, I'll explain main kubernetes component you need to know.

## Kubernetes Pods

A Pod is the fundamental building block in the Kubernetes. Usually, a pod runs only a single container.  But there are some instances you need to run more than one container in the pod. Such as init containers and internal containers.

* This example defines pod manifest for single containers

```yaml
---
# Digital Avenue Pod Definition File
kind: Pod
apiVersion: v1
metadata:
  name: nginx-app
  labels:
    app: nginx
spec:
  containers:
    - name: nginx-app
      image: nginx
```

* This example defines pod manifest with "InitContainers."

```yaml
---
# Digital Avenue Pod Definition File
kind: Pod
apiVersion: v1
metadata:
  name: nginx-app
  labels:
    app: nginx
spec:
  containers:
    - name: nginx-app
      image: nginx
      volumeMounts:
      - mountPath: "/etc/nginx"
        name: nginx-pv-claim
  initContainers:
    - name: volume-mount-data-log
      image: busybox
      imagePullPolicy: IfNotPresent
      command: ["sh", "-c", "chown -R 101:101 /etc/nginx"]
      volumeMounts:
      - mountPath: "/etc/nginx"
        name: nginx-pv-claim
```

The above configuration shows the Nginx container with a busy-box init container. In here, the busybox container start first and making sure correct permission "UID":101 and "GID":101 in the "/etc/nginx" persistent volume. Then spin up the nginx container secondly.

## Kubernetes Replication Controller

If you deploy only a pod, It is not possible to scale and replicate. Once you delete it, it's gone. Replication controllers maintain a minimum number of pods that are always running. To do this, we need to have an object called "ReplicationController". However, Now Kubernetes has introduced a new object called "ReplicaSet" as a replacement for "ReplicationController."


```yaml
# Digital Avenue Replication Controller Definition File
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx-app
        image: nginx
```

In the above Kubernetes manifest, The replicas determine the number of pods that would run at their desired state. Replication Controllers scales pods by searching on the pod labels. "Selector" labels use to match the pod labels exactly.

## Kubernetes Replica Sets

Replica Sets are the new replacement for the Replication Controllers.
Unlike the Replication Controllers, The ReplicaSets can use set-based search notations to group pods rather than a named Key: Value pair based match. You will get more control and more dynamic selector options to use ReplicaSet.

```yaml
# Digital Avenue ReplicaSet Definition File
apiVersion: extensions/v1beta1
kind: ReplicaSet
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx 
      environment: dev
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
        environment: dev
    spec:
      containers:
      - name: nginx-app
        image: nginx
```

Or like this...

```yaml
# Digital Avenue ReplicaSet Definition File
apiVersion: extensions/v1beta1
kind: ReplicaSet
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchExpressions:
      - {key: app, operator: In, values: [nginx]}
      - {key: environment, operator: NotIn, values: [production]}
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
        environment: dev
    spec:
      containers:
      - name: nginx-app
        image: nginx
```

## Kubernetes Deployments

Deployment manifest is the most recommended and primarily used method to deploy Kubernetes pods. Deployment manifest replaces "Replication Controllers". Deployment is the most powerful Kubernetes object since It's possible to roll out and roll back changes/deployments. 

You can use deployments when...

* You require scaling and self-healing of your pods

* If you are running stateless applications that are don't required to store persistently on a disk, such as applications connected with back-end services and databases. 


```yaml
# Digital Avenue Deployment Definition File
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx 
      environment: dev
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
        environment: dev
    spec:
      containers:
      - name: nginx-app
        image: nginx
```

Deployment manifest also performs the same task as ReplicaSet. Deployments manage pods employing a ReplicaSet. The "Deployment" manifest pretty much slimmer to  "ReplicaSet". The only difference is the only object type.

### Scaling Deployments

Scaling Deployment can be performed in declarative and imperative ways. Which means you can modify the replica section of the manifest file and run it. 

```bash
"kubectl apply -f <manifest file>"
```


Or else, Run this command imperatively.

```bash
"kubectl scale deployment nginx-deployment --replicas=10"
```

But, modifying the manifest file is the most recommended way.

### AutScaling Deployments

Kubernetes cloud auto-scale the deployment if the pod exceed the defined threshold.  To perform this, Kubernetes uses the object called "HorizontalPodAutoScaler". This object checks the pod metrics, and if it breaches a defined threshold, It spins up a new pod to maintain the load. So, you need to specify the minimum and the maximum number of pods.

#### Horizontal Pod AutoScaler

The Horizontal Pod AutoScaler ensures your Kubernetes deployment ca scale based on pre-defined metrics horizontally. Kubernetes also can manage the creation and deletion of pods based on metrics. 

You can do this either using "kubectl" or using "HorizontalPodScaler" manifest file.

First of all, we need to modify our deployment manifest to apply a resource limit to the pods. 

```yaml
# Digital Avenue Deployment Definition File
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx 
      environment: dev
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
        environment: dev
    spec:
      containers:
      - name: nginx-app
        image: nginx
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 200m
```

This configuration ensures the nginx container limit up-to 500 milicore of the worker node.


#### Using "HorizontalPodAutoScaler" 

You can auto-scale pods using more advanced metrics and custom metrics such as network utilization.

```yaml
# Digital Avenue HorizontalPodAutoscaler Definition File
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  - type: Pods
    pods:
      metric:
        name: packets-per-second
      target:
        type: AverageValue
        averageValue: 1k
  - type: Object
    object:
      metric:
        name: requests-per-second
      describedObject:
        apiVersion: networking.k8s.io/v1beta1
        kind: Ingress
        name: main-route
      target:
        type: Value
        value: 10k
```

According to the above manifest, you can see there are three kinds of metrics. 
* Resource Metrics: CPU, Memory
* Pod Level Metrics: Metrics such as "Packet-Per-Second" and  other network traffic within the pod
* Object Level Metrics: Metrics such as "Requests-Per-Second" and other network traffic in other Kubernetes objects such as Ingress Controllers.

## Managing Stateful Applications

So far, We were discussing stateless applications. How are We going to take care of our Stateful applications that need to persist their data on a disk even after a reboot? Imagine a front-end application that has a database. 

Kubernetes offers many  kinds of objects to manage Stateful applications using "StatefulSet", "StorageClass", "PersistentVolumeClaim" and "PersistentVolumes"

We will discus other most commonly used Kubernetes object in the next episode.
