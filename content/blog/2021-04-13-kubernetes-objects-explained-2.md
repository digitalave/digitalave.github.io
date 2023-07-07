---
title: 'Most Wanted Kubernetes Objects Explained - Episode 02'
subtitle: Stateful Sets, Persistent Volumes, Persistent Volume Claims, Storage Class, Daemon Sets, Services, Ingress Controllers
description: "KSo far, We were discussing about the stateless applications. But, How are We going to take of our Stateful applications that need to persist their data on a disk even after a reboot. Imagine a front-end application which has a database."
image: /img/post-imgs/k8s-objects-02/k8s-objects-explained-02.jpg
categories: ["DevOps"]
type: "regular" # available types: [featured/regular]
draft: false
date: 2021-04-13
---

## Managing Stateful Applications

So far, We were discussing about the stateless applications. But, How are We going to take of our Stateful applications that need to persist their data on a disk even after a reboot. Imagine a front-end application which has a database. 

Kubernetes offers many  kinds of objects to manage Stateful applications using "StatefulSet", "StorageClass", "PersistentVolumeClaim" and "PersistentVolumes"

## Stateful Sets

When you running a Stateful application using dynamic volume provisioning, Kubernetes ensure that storage was mounted at a volume to the pod. We need to mention our Kubernetes service in the "StatefulSet" manifest. That's why we need to define the "serviceName" in the "StatefulSet" spec section.


A StatefulSet also requires a "PersistentVolume"  which can provisioned either manually or through an automatic provisioned  through a storage class. This is why we mention "VolumeClaimTemplate" in the "StatefulSet" manifest.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class"
      resources:
        requests:
          storage: 1Gi
```

## Persistent Volume (PV)

Persistent volumes are storage resources such as disks and file systems. They can provisioned either manually by administrator or dynamic provisioning using Storage Classes.
The we can mount these disks into containers running inside pods to ensure the data persistence.

Kubernetes has operational activities such as Provisioning, Configuring and attaching. These relationships can be mapped as shown below.

| Storage Activity | Storage Primitive |
|------------------|-------------------|
| Provisioning     | Persistent Volume |
| Configuring      | Storage Class     |
| Attaching        | Persistent Volume Claim |

### Static Provisioning 

In static provisioning, The administrator needs to create volumes manually and supply those volume information into a persistent volume manifest. In order to use these Volumes needs be claimed. 

IMG5

In this manifest shows how we can provision our persistent volume using Azure Disk. Also we need to include persistent volume claim template.

```yaml
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nginx-pv
  labels:
    usage: nginxdisk
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  azureDisk:
    kind: Managed
    diskName: myAKSDisk
    diskURI: /subscriptions/<subscriptionID>/resourceGroups/myRG/providers/Microsoft.Compute/disks/myAzureDisk
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 2Gi
      selector:
        matchLabels:
          usage: nginxdisk
```

### Dynamic Volume Provisioning

Even though the static provisioning is a good option, in some cases we may have to shift our workload into a different cloud provider. Then we may have to change our volume information in the manifest file according to the newly provisioned volumes. So... what if you have hundreds or thousands of containers to mount in this way? So, this may not be the good option at all.


To ensure a more dynamic and platform independent solution, Here we would create a storage class object which define the kind of object such as Disk, SSD, NFS Share, Block Storage, etc. finally, the cloud provider is going to provision it for us. Then PersistentVolumeClaim use this storage class to tell the cloud provider to provision a storage. Finally, Kubernetes mount the PersitentVolumeClaim (PVC) as a volume to the Pod.

## Storage Class

A Storage Class object defines the kind of object which needs provisioning ( Disks, SSD, Block Storage, NFS...). Then, the cloud provider is going to provision the volume. In here storage class name should be platform independent ( Fast, Disk, SSD...)

In this example, Here we provisioning a Azure Managed Premium Storage in a "fast" Storage Class.

```yaml
---
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: fast
provisioner: kubernetes.io/azure-disk
reclaimPolicy: Retain
parameters:
  storageaccounttype: Premium_LRS
  kind: Managed
```

## Persistent Volume Claim (PVC)

Persistent Volume Claim refers the storage classes dynamically. It can provision volumes as required. In this example, storage class is referring as fast storage class to provision an Azure disk of 5GB. Then, volumes can mount to the container using "PersistentVolumeClaim"

```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: azure-managed-disk
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: fast
  resources:
    requests:
      storage: 5Gi
```

### Dynamic Provisioning On a StatefulSet

When we at the manifest carefully. You can see there is no singple place that we mentioned a cloud provider. In this way, we can make storage provisioning is more portable and independent from cloud providers.


```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "fast"
      resources:
        requests:
          storage: 5Gi
```

## DaemonSet

Daemons are background processes that are specialized to perform specific admin tasks such as collect logs, monitoring and more.

Kubernetes object called "DaemonSets"" are using special use cases such as monitoring node health, shipping logs, and spinning up cluster storage daemons.

In this DaemonSet manifest shows which runs FluentD logging app within all nodes of the Kubernetes cluster.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      # this toleration is to have the daemonset runnable on master nodes
      # remove it if your masters can't run pods
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

## Exposing Kubernetes Applications

In this section, I'm going to discuss the multiple ways to expose Kubernetes objects to internal or external clients through the Kubernetes services and Ingress resources.

## Services

Pods are ephemeral objects. Which means, Pod's lifespan is end with restarts. Every Pod has an IP address. If one unhealthy pod dies, Kubernetes replace it with fresh pod with different IP address. This leads to a problem as we cannot use the same IP address which was in previous pod.

To overcome this, Kubernetes provides a built-in service discovery mechanism called CoreDNS. CoreDNS can keep track of the IP addresses of the Pods. This CoreDNS service maintain a DNS register which helps to provide internal Load Balancing and DNS Resolution. Therefore, We don't need have to worry about the dynamic IP addresses of Pods. 

Based on this mechanism we can expose our application in three ways.

* ClusterIP
* NodePort
* LoadBalancer

## ClusterIP Services

ClusterIP service is the default service type. Let's say something we are running a database in a container within Kubernetes cluster. And we don't need to expose our database pod outside the Kubernetes cluster. In here ClusterIP service is discoverable only within the Kubernetes cluster.

But any pods within the cluster can access the database using its "ServiceName:Port" which can routed traffic into the correct pod through dynamic service discovery. And also  we don't need to worry about pod's IP. 

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

## NodePort Services

A NodePort service exposes your Pods into node's static port called the NodePort. This NodePort can discoverable from any of Kubernetes nodes. If your nodes reachable from the internet, external clients would able to access your pod on any "NodeIP:NodePort"

These NodePort ranges from 30000 - 32767
This mechanism is not recommended for production deployments.

In this example shows NodePort service expose the nginx deployment on Port 31000. If you don't specify a port in the manifest, it pickup freely available port in the range.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80 
      nodePort: 31000
  type: NodePort
```

## LoadBalancer Services
In general, a load balancer service generates a NodePort and requests that the cloud provider automatically provision a load balancer in front of your Nodes.

To clarify, suppose you have a master and three worker nodes called master, node01, node02, and node03, as well as an NGINX pod running on port 80 that you want to open to outside world.

A LoadBalancer service will first create a NodePort on any random port from the NodePort range (assume,31000), and also a Load Balancer with master:31000, node01:31000, node02:31000, and node03:31000 as the backend.

In a production system, a Load Balancer service is one of the ways to expose your frontend pods to the Internet inside of your internal infrastructure.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80 
  type: LoadBalancer
```
