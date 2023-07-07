---
title: 'How To Run Microsoft SQL Server On Kubernetes - Azure Kubernetes Service'
subtitle: Highly Available Microsoft SQL Server on AZure AKS
description: "Setup a SQL Server instance on Kubernetes with persistent storage for high availability in Azure Kubernetes Service (AKS). If the SQL Server instance dies, Kubernetes re-creates it in a new pod automatically. "
image: /img/post-imgs/sql-on-aks/MSSQL-On-AKS.jpg
categories: ["DevOps"]
type: "featured" # available types: [featured/regular]
draft: false
date: 2021-04-17
---

#### Prerequisites:

* Azure CLI

    https://docs.microsoft.com/en-us/cli/azure/install-azure-cli

### 1. Run the Azure CLI with the az command.

### 1.1 Run the login command.
		
```bash
az login
```

Login in the browser with the azure account.
		
### 2. Activate the correct subscription. 
    
Azure uses the concept of subscriptions to manage spending. You can get a list of subscriptions your account has access to  by running:


```bash
az account list --refresh --output table
```

Choose correct subscription (If it has more than one)

* SubscriptionId : cfg55fb3-ck7f-4342-6a1a-f934d2254ca7

* Name : DigitalAvenue azure subscription 

### 2.1 Select Subscription

Pick the subscription you want to use for creating the cluster, and set that as your default. If you only have one subscription you can ignore this step.

```bash
az account set -s <YOUR-CHOSEN-SUBSCRIPTION-NAME>

az account set -s cfg55fb3-ck7f-4342-6a1a-f934d2254ca7
```

### 3. Create a Resource Group

### 3.1 Choose location Cluster has to be resided
		
```bash
az account list-locations --query '[].name'
```

OR 

```bash
az account list-locations --query "sort_by([].{DisplayName:displayName, Name:name}, &DisplayName)" --output table
```

List location from above command
    
### 3.2 Create the Resource group

```bash
az group create --name DigitalAvenueRSG --location uksouth
```

For eg:

```bash
az group create --name <RESOURCE_GROUP_NAME> --location <AZ Location>
```

### 4. Create AKS cluster

### 4.1 Generate ssh key pair

```bash
ssh-keygen -f ssh-key-<CLUSTER-NAME>
```

For eg: 
    
```bash
ssh-keygen -f ssh-key-DigitalAvenue
```

### 4.2 Download and install kubectl and kubelogin

```bash
az aks install-cli
```

Note: Follow the guidelines with the command output

### 4.3 Deploy K8S Cluster on AKS

Choosing VM Type: 
    
References: 

[https://docs.microsoft.com/en-us/azure/virtual-machines/sizes-b-series-burstable][1]

[https://azure.microsoft.com/en-us/pricing/details/virtual-machines/ubuntu-advantage-standard/][2]


This process may take few minutes. Please wait until it completed.

```bash
az aks create --name DigitalAvenue-Cluster --resource-group DigitalAvenueRSG --ssh-key-value .\ssh-key-DigitalAvenue.pub --node-count 1 --vm-set-type VirtualMachineScaleSets --load-balancer-sku standard --enable-cluster-autoscaler --min-count 1 --max-count 2 --node-vm-size Standard_B4ms --output table
```

For eg:

```bash
az aks create --name <CLUSTER-NAME> \
    --resource-group <RESOURCE_GROUP_NAME> \
    --ssh-key-value ssh-key-<CLUSTER-NAME>.pub \
    --node-count 1 \
	--vm-set-type VirtualMachineScaleSets \
	--load-balancer-sku standard \
	--enable-cluster-autoscaler \
	--min-count 1 \
	--max-count 2 \
    --node-vm-size <VM Type> \
    --output table
```


### 5. Connect to cluster

### 5.1 Connect to AKS cluster using following command (Authentication)

```bash
az aks get-credentials --resource-group <Resource-Group-Name> --name <AKS-Cluster-Name>
```

For eg:

```bash
az aks get-credentials --resource-group DigitalAvenueRSG --name DigitalAvenue-Cluster
```

5.2 Verification

```bash
kubectl get nodes
```

This command will returns the created cluster node name and status

#### Deploying SQL Server Container in Kubernetes with AKS

### 6. Manage AKS Cluster

Available Features:

* High Availability
* Data Persistency - (Even after a DB crash event)
* Resiliancy - If SQL Server fails, kubernetes automatically recreate DB in a new pod

## 6.1 Create a namespace

```bash
vim DigitalAvenue-ns.yaml
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: DigitalAvenue
```

```yaml
kubectl apply -f DigitalAvenue-ns.yaml
```

### 6.2 Create SA Password - Secret Password For SQL

SA manages sensitive information such as DB passwords and configurations.

Provide secure passwrd before creating a SA secret

```bash
kubectl create secret generic mssql --from-literal=SA_PASSWORD="<Secure-Password>" -n <NameSpace>
```

For eg:

```bash
kubectl create secret generic mssql --from-literal=SA_PASSWORD="PaSsW@rD" -n DigitalAvenue
```

### 8. Create a Persistent Storage

### 8.1 Create "PersistentVolumeClaim" and "PersistentVolume"

```bash
vim db-pvc.yaml
```

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
     name: azure-disk
     namespace: DigitalAvenue
provisioner: kubernetes.io/azure-disk
parameters:
  storageaccounttype: Standard_LRS
  kind: Managed
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mssql-data
  namespace: DigitalAvenue
  annotations:
    volume.beta.kubernetes.io/storage-class: azure-disk
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 15Gi
```

Create PersistentVolumeClaim

```bash
kubectl apply -f .\db-pvc.yaml -n DigitalAvenue
```

verify and check status 

```bash
kubectl get pv,pvc -n DigitalAvenue
```
   
```bash
kubectl describe pvc mssql-data -n DigitalAvenue
```

### 9. Create the deployment

MSSQL Container based on following Docker image: 
    [https://hub.docker.com/_/microsoft-mssql-server][3]

  [1]: https://hub.docker.com/_/microsoft-mssql-server
  [2]: https://hub.docker.com/_/microsoft-mssql-server
  [3]: https://hub.docker.com/_/microsoft-mssql-server

### 9.1 Create Kubernetes manifest for the MSSQL deployment. 

```bash
vim db-deployment.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mssql-deployment
spec:
  replicas: 1
  selector:
     matchLabels:
       app: mssql
  template:
    metadata:
      labels:
        app: mssql
    spec:
      terminationGracePeriodSeconds: 30
      hostname: mssqlinst
      securityContext:
        fsGroup: 10001
      containers:
      - name: mssql
        image: mcr.microsoft.com/mssql/server:2019-latest
        ports:
        - containerPort: 1433
        env:
        - name: MSSQL_PID
          value: "Developer"
        - name: ACCEPT_EULA
          value: "Y"
        - name: SA_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mssql
              key: SA_PASSWORD 
        volumeMounts:
        - name: mssqldb
          mountPath: /var/opt/mssql
      volumes:
      - name: mssqldb
        persistentVolumeClaim:
          claimName: mssql-data
---
apiVersion: v1
kind: Service
metadata:
  name: mssql-deployment
spec:
  selector:
    app: mssql
  ports:
    - protocol: TCP
      port: 1433
      targetPort: 1433
  type: LoadBalancer
```

##### Environment variables

>    MSSQL_PID: 
>         Edition : Developer  (Enterprise, Standard, or Express)
>     
>    ACCEPT_EULA:
>         License Agreement : Y
>         
>    SA_PASSWORD:
>         MSSQL Password Secret : [Added at the step 6.1 ]


##### Service

Network Service: loadbalancer

Apply deployment

```yaml
kubectl apply -f db-deployment.yaml -n DigitalAvenue
```

### 9.2 Verify the Deployment 

```bash
kubectl get pv,pvc,pod,deployment,svc,secret -n DigitalAvenue
```

Execute below command to get more information about the cluster. It will automatically redirect you to Azure Portal. 

```bash
az aks browse --resource-group <MyResourceGroup> --name <MyKubernetesClustername>

az aks browse --resource-group DigitalAvenueRSG --name DigitalAvenue-Cluster
```

### 9.3 Connect to the SQL Server instance

Using "sqlcmd" CLI

```bash
sqlcmd -S <External IP Address> -U sa -P "PaSsW@rD"
```

