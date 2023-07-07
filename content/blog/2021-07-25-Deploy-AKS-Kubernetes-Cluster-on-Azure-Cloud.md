---
title: "Deploy Production Grade Kubernetes Cluster on Azure AKS"
# image: /img/post-imgs/docker-getting-started/docker.jpg
categories: ["DevOps"]
type: "regular" # available types: [featured/regular]
draft: false
date: 2021-07-25
---

## Introduction
  
This tutorial is intended to demonstrate how to setup your 1st Kubernetes cluster on Azure Kubernetes Services (AKS). This tutorial will cover up all the steps that you need to setup complete AKS cluster.
You can continue alone with me without having major knowledge of Docker, Kubernetes or Azure AKS.

As you already may know, Kubernetes provides a distributed infrastructure for containerized application on  the Azure Cloud. 

Azure Kubernetes Service (AKS) is a Managed service which provides by Microsoft Azure. Microsoft taking care of the lot's of things such high availability, scalability, monitoring and maintenance. By using with Azure Kubernetes Services, you can deploy production ready highly available Kubernetes cluster in few minutes.

######  Deploy Production Grade Kubernetes Cluster on Azure AKS (Zero to Hero Guide)


### Learning Goals...

* Create an Azure Kubernetes Service (AKS)

* Create an Azure Container Registry (ACR)

* Attach ACR with AKS

* Integrate AKS with Azure Active Directory (AAD)
    (Managing AKS Cluster Using Azure AD Users/Groups)

* Configure AKS Cluster for future Windows Node Pools

## Before You Begin !!!

I created some commands composed using windows Powershell on Visual Studio Code IDE. Therefore, Some variable assignment and next line syntax types may not be compatible on Linux bash terminals. But, You can change them very easily.

For Example:
* Variable assignment
Powershell variable Assignment:

```powershell
$AKS_RESOURCE_GROUP = "DigitalAvenue-RSG"
```

Linux Variable Assignment:

```bash
AKS_RESOURCE_GROUP="DigitalAvenue-RSG"
```


* Next line statement.

If you are using a Linux terminal, Please replace Powershell "`" next line syntax with "\" Linux next line syntax.

For ex:

Powershell Command:

```powershell
az ad group show `
    --group AKS-Admins `
    --query objectId `
    --output tsv
```


Linux Bash Command:

```bash
az ad group show \
    --group AKS-Admins \
    --query objectId \
    --output tsv
```

I'm pretty sure you've got the point. okay let's jump !!!

## Deployment Workflow

1. Installing prerequisites tools
   
   (Docker Desktop, Azure CLI, KubeCTL)
   
2. Login to Azure / Set Subscription
   
3. Create Resource Group
 
4. Azure AD User/Group Integration with AKS
 
 This allows you to define who can have access to the Azure AKS
 
    REF: https://docs.microsoft.com/en-us/azure/aks/azure-ad-rbac
    
5. Create a SSH Key 

6. Create Azure Container Registry (ACR)

7. Create AKS Cluster with System(Linux) Node Pool

## Deployment Prerequisites:

* Azure Subscription ($200 free credit)
As the initial requirement, you should have a Microsoft Azure Account. You can get a free Azure account with $200 worth credits for one year.

REF: https://azure.microsoft.com/en-us/free/

* Docker Desktop (Windows/Linux/Mac)
Installing Docker in local machines a primary requirement for Kubernetes testing and deployments.

    REF: https://www.docker.com/products/docker-desktop

* Azure CLI
Then, you need to install Azure CLI on your local machine. Azure CLI is available to install in Windows, Mac and Linux environments.

    REF: https://docs.microsoft.com/en-us/cli/azure/install-azure-cli

* Kubectl
Kubectl is the Kubernetes command line tool which allows us to execute commands against the Kubernetes cluster.

    REF: https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/

Before we continuing the AKS Cluster deployment,  We need to complete the following prerequisites. 

## STEP 01: Install Prerequisites Tools

As the 1st step I need to install following tools in order to continue the deployment process. So, I'm quickly go through with the required packages installation. 

**If they are already available on your computer, then directly jump into the STEP-02**

* Install Docker Desktop

    You can simply download and install docker Docker Desktop from <a href="https://www.docker.com/products/docker-desktop" target="_blank">here</a>.

* Install Azure CLI

    There are several ways to install Azure CLI on your computer. I'm this I'm going to install Azure CLI package using this powershell command.
    
    Open up you Windows powershell as an **administrator** and then run the command below.
    
```powershell
Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi; Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'; rm .\AzureCLI.msi
```

Or else, you also can install Windows MSI binary using this <a href="https://aka.ms/installazurecliwindows" target="_blank">link</a>. 


Ref: <a href="https://docs.microsoft.com/en-us/cli/azure/install-azure-cli" target="_blank">https://docs.microsoft.com/en-us/cli/azure/install-azure-cli</a>


* Install KubeCTL

There are many ways to install KubeCTL on windows. Either you can install using "curl" or using "Chocolatey" package manager.

**Option 01: Install Using Curl:** 

Make sure whether the "curl" package available on your computer.
    
```powershell
curl -LO https://dl.k8s.io/release/v1.21.0/bin/windows/amd64/kubectl.exe
```

Once you downloaded, move "kubectl.exe" binary into somewhere your programs resides. And make sure to set the **PATH** environment variable for "kubectl.exe"

**Option 02: Installing  Using Chocolaty Package Manger:**

Open up Powershell as an administrator and simply execute the following commands.

Install Chocolaty:

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```

```powershell
choco install kubernetes-cli
```

```powershell
kubectl version --client
```

```powershell
mkidr ~/.kube
cd ~/.kube
New-Item config -type file
```

Ref: <a href="https://chocolatey.org/install" target="_blank">https://chocolatey.org/install</a>

## STEP 02: Login and Set Subscription

### 2.1 Login to Azure CLI

Before moving further, we need to authenticate with Azure using the following command. 1st and foremost we need to login into our Azure subscription. Hit the following commend to login into Azure account.

```powershell
az login
```

After entering the above command, you web browser will open and ask for relevant credentials for Azure Portal.
		
### 2.2 Activate the correct subscription. 
    
Azure uses the concept of subscriptions serve as a single billing using for azure resources and services among different environments such as dev, staging and prod. If you have more than one subscription with you account, You can run the following command to list all the subscriptions associated with your Azure account and choose one for your deployment.

List Subscriptions:

```powershell
az account list --refresh --output table
```

Set Subscription:

Pick the subscription you want to use for creating the cluster, and set that as your default. If you only have one subscription you can ignore this step.

```powershell
az account set -s $SUBSCRIPTION_ID
```

## 3. Create a Resource Group

### 3.1 Choose location Cluster has to be resided

And then you need to define where you going to deploy your AKS cluster. For that, you can choose a Azure data-center which is close to you. This command list all the data-centers with their location.

```powershell
az account list-locations --query "sort_by([].{DisplayName:displayName, Name:name}, &DisplayName)" --output table
```

### 3.2 Create the Resource group

Further moving one, We need to create a separate resource group. This is where all the cluster resources going to create. Azure Resource Groups are logical collection, which can holds related Azure services and resources almost anything in Azure. This resource groups allows you to manage every relevant resources in a separate environment.

> --name = Resource Group Name
> 
> --location = Azure Data-Center Location

Okay, Let's create a resource group. And keep in mind to create all the AKS cluster specific resources within this resource group in same region/location.

```powershell
az group create --name $AKS_RESOURCE_GROUP --location $AKS_REGION
```

## STEP 04: Create SSH Key

In some cases, You may need to login into an AKS Node for the maintenance activities such as logs collections or even troubleshooting activities. If something happens in future we might require SSH credentials.

Okay, Simply hit the following commands to create a SSH Key.

```powershell
mkdir $HOME/.ssh/aks-prod-sshkeys/
```

```powershell
ssh-keygen -t rsa -C "HOSTNAME" -f "$HOME/.ssh/aks-prod-sshkeys/aksprod-id_rsa"
```

Set SSH Key path into a environment variable.

```powershell
$AKS_SSH_KEY_LOCATION="$HOME/.ssh/aks-prod-sshkeys/aksprod-id_rsa.pub"
```

## STEP 05: Integrate Azure AD with AKS (AKS-managed Azure Active Directory integration)

There are numerous ways to secure your Azure Kubernetes Cluster. We need to  thoroughly consider about the AKS security, since we are going to deploy a production ready cluster. In that sense, I'm going to integrate Azure Active Directory with Azure Kubernetes Services (AKS). Therefore we can authenticate, control access and authorize from Azure AD to AKS Cluster securely.

AKS can be configured to use Azure AD for user authentication, We can can use Kubernetes Role-Based Access Control (RBAC) to limit access to  the cluster resources.

#### Limitations
* AKS-managed Azure AD integration can't be disabled
* non-Kubernetes RBAC enabled clusters aren't supported for AKS-managed Azure AD integration
* Changing the Azure AD tenant associated with AKS-managed Azure AD integration isn't supported
* Changing a AKS-managed Azure AD integrated cluster to legacy AAD is not supported


### 5.1 Create Azure AD Group

Now, I'm going to create a AKS cluster administrator group.

$AD_AKS_ADMIN_GROUP = "aksadmins"

Following command creates a group in Azure AD for the AKS cluster administrators. And assigned the group id into a variable.

```powershell
az ad group create --display-name AKS-Admins --mail-nickname $AKS_ADMIN_GROUP
```

```powershell
$AD_AKS_ADMIN_GROUP_ID=$(az ad group show --group AKS-Admins --query objectId --output tsv)
```

Or else, you can run this single command to create admin group and get the objectID for the group at once. 

```powershell
$AD_AKS_ADMIN_GROUP_ID=$(az ad group create --display-name AKS-Admins --mail-nickname $AKS_ADMIN_GROUP --query objectId -o tsv)
```

### 5.2 Create Azure AD AKS Admin User 

Then, I'm going to create "aksadmin1" user and assign user ID into  the variable

```powershell
az ad user create --display-name "AKSAdmin1" --password "DAvenue@12345" --user-principal-name aksadmin1@DigitalAvenue105.onmicrosoft.com
```

```powershell
$AD_AKS_ADMIN1_USER_OBJECT_ID=$(az ad user show --id aksadmin1@DigitalAvenue105.onmicrosoft.com --query objectId --output tsv)
```

Or else, you can run this single command to create admin user and get the objectID for the user at once. 

```powershell
$AD_AKS_ADMIN1_USER_OBJECT_ID=$(az ad user create `
  --display-name "AKSAdmin1" `
  --user-principal-name aksadmin1@DigitalAvenue105.onmicrosoft.com `
  --password "DAvenue@12345" `
  --query objectId -o tsv)
```

**NOTE:** Please make sure to replace "aksadmin1@DigitalAvenue105.onmicrosoft.com" with your AD Domain with in "user-principal-name"

### 5.3 Associate "AKS-Admin" User to "AKS-Admins" Group

```powershell
az ad group member add --group AKS-Admin --member-id $AD_AKS_ADMIN1_USER_OBJECT_ID
```

**Note:** Make sure to note down your "username" and "password".

> UserName : aksdmin1@DigitalAvenue105.onmicrosoft.com
>
> Password : DAvenue@12345


## STEP 06: Get Azure AD Tenant ID and Set Windows Username Password

In future, You may need to run Windows based containers on your Kubernetes cluster. But, Generally, Windows containers (such as Windows Server 2019) cannot be run on Linux Nodes. So, Over the time you can add  a windows node pool into your AKS cluster. So, I'm going getting ready for that as well.

### 6.1 Get Azure AD Tenant ID

You find  your tenant ID either using Azure Portal or Azure CLI. 

```powershell
$AZURE_DEFAULT_AD_TENANTID=$(az account show --query tenantId --output tsv)
```

Or, Head-over to Azure Portal and go to Services -> Azure Active Directory -> Properties -> Tenant ID

### 6.2 Set Username and Password for Windows Node

$AKS_WINDOWS_NODE_USERNAME= "aksuser"
$AKS_WINDOWS_NODE_PASSWORD= "DAvenue@123456"   # Required length: [14 -123]

## STEP 07: Create Azure Container Registry (ACR)

Then, We need to create an Azure Container Registry (ACR). ACR is a private container registry which allows you to store and manage container images securely. 

**Note:** The name of the ACR must be globally unique. choose something not exist already.
Run the following line to create an Azure Container Registry if you do not already have one

Check if your desired name is available

$ACR_NAME = "DAvenueACR"

```
az acr check-name -n $ACR_NAME
```

```powershell
az acr create --resource-group $AKS_RESOURCE_GROUP --name $ACR_NAME --sku Standard --location $AKS_REGION
```

## STEP 08: Create AKS Cluster With System/Linux Node Pool

Now, I'm going to  deploy AKS cluster with following parameters and their variables. While you are doing, make sure to assign correct parameter as I've done here. 

Run  below command to create AKS cluster.


Here we'll pass following variables.

```powershell
$SUBSCRIPTION_ID = "9e327125-90eb-4bb9-9f0f-069d133bed59"
$AKS_RESOURCE_GROUP = "DigitalAvenue-RSG"
$AKS_CLUSTER_NAME = "DAvenueAKS-Cluster"
$AKS_REGION = "southeastasia"
$AKS_SSH_KEY_LOCATION="$HOME/.ssh/aks-prod-sshkeys/aksprod-id_rsa.pub"
$AKS_ADMIN_GROUP = "aksadmins"
$AKS_ADMIN_USER1 = "aksadmin1"
$AD_AKS_ADMIN_GROUP_ID=$(az ad group show --group AKS-Admins --query objectId --output tsv)
$AD_AKS_ADMIN1_USER_OBJECT_ID=$(az ad user show --id aksadmin1@DigitalAvenue105.onmicrosoft.com --query objectId --output tsv)
$AZURE_DEFAULT_AD_TENANTID=$(az account show --query tenantId --output tsv)
$ACR_NAME = "DAvenueACR"
$AKS_WINDOWS_NODE_USERNAME= "aksuser"
$AKS_WINDOWS_NODE_PASSWORD= "DAvenue@123456"   # Required length: [14 -123]
$LinuxNodeVMSize = 'Standard_B2s'
```


Let's create our AKS cluster with Azure AD enabled.

```powershell
az aks create --resource-group ${AKS_RESOURCE_GROUP} `
              --name ${AKS_CLUSTER_NAME} `
              --enable-managed-identity `
              --ssh-key-value  ${AKS_SSH_KEY_LOCATION} `
              --admin-username aksnodeadmin `
              --node-count 1 `
              --enable-cluster-autoscaler `
              --min-count 1 `
              --max-count 2 `
              --network-plugin azure `
              --enable-azure-rbac `
              --enable-aad `
              --aad-admin-group-object-ids ${AD_AKS_ADMIN_GROUP_ID} `
              --aad-tenant-id ${AZURE_DEFAULT_AD_TENANTID} `
              --windows-admin-password ${AKS_WINDOWS_NODE_PASSWORD} `
              --windows-admin-username ${AKS_WINDOWS_NODE_USERNAME} `
              --node-osdisk-size 30 `
              --attach-acr ${ACR_NAME} `
              --node-vm-size ${LinuxNodeVMSize} `
              --nodepool-labels nodepool-type=system nodepoolos=linux app=davenue-apps `
              --nodepool-name systempool `
              --nodepool-tags nodepool-type=system nodepoolos=linux app=davenue-apps `
              --enable-ahub `
              --location ${AKS_REGION}
```

--enable-aad = AKS cluster with managed Azure AD integration - enable administrative access to Azure AD Group
--enable-azure-rbac = Azure RBAC for Kubernetes Authorization

IMG1

AKS cluster creation will take up-to 5 - 10 minutes. Wait until it completed. Once you have completed all the above step, you can see numerous resources had been created on your AKS resource group.

IMG2

IMG8

### STEP 09: Create AKS Role Assignment To Azure AD User / Group

In this step, I will assign "AKS Cluster Admin Role" into previously created AKS-Admins group. (STEP 3.3)

There are two build-in Azure RBAC roles available for AKS. 

* AKS Cluster Admin Role
* AKS Cluster User Role

Refer fore more info: https://docs.microsoft.com/en-us/azure/aks/control-kubeconfig-access#available-cluster-roles-permissions

In this case I'm allowing super-user access to "AKS-Admins" group. Therefore, Users who are in the "AKS-Admins" group  will gain full control over every resource in the cluster and in all namespaces You can change it according your requirement. And this Roles assignments scoped to the entire AKS cluster.

We need to have Azure AD group ID and AKS Cluster ID,  In order to  assign this cluster admin role to the Azure AD group.

Already  We got the AKS-Admins group object ID in the step 3.1.

```powershell
$AD_AKS_ADMIN_GROUP_ID=$(az ad group show --group AKS-Admins --query objectId --output tsv)
```

# Get your AKS Resource ID

```powershell
$AKS_ID=$(az aks show -g $AKS_RESOURCE_GROUP -n ${AKS_CLUSTER_NAME} --query id -o tsv)
```

Here you can assign Cluster Admin Role to your user or whole group.

# Assign the 'Cluster Admin' role to the User 

```powershell
az role assignment create `
    --assignee $ACCOUNT_ID `
    --scope $AKS_ID `
    --role "Azure Kubernetes Service RBAC Admin"
```

Or else, 

# Assign the 'Cluster Admin' role to the User 

```powershell
az role assignment create --role "Azure Kubernetes Service RBAC Admin" `
    --assignee-object-id ${AD_AKS_ADMIN_GROUP_ID} `
    --assignee-principal-type Group `
    --scope $AKS_ID
```

--assignee-principal-type = {Group, ServicePrincipal, User}

IMG9

### STEP 10: Configure Credentials and Access Azure AD Enabled AKS Cluster

In order to authenticate with your AKS cluster, You need to download the kubeconfig file into your local machine.

```powershell
az aks get-credentials --resource-group $AKS_RESOURCE_GROUP --name $AKS_CLUSTER_NAME --admin
```

IMG3 

### STEP 11: Verification

Now, The cluster creation is almost completed. But, We can verify it using following commands.

As the first step, I'm going to check whether out Azure AD authentication is working. Simply, hit the following command on your terminal

IMG4

#### Get AKS Nodes

```powershell
kubectl get nodes
```

It will ask you to verify your account activity with device login access token. Please follow the command output and open the URL and paste your TOKEN. Output will look like this once you authenticated successfully.

```powershell
PS C:\Users\dimui\deployments> kubectl get nodes

To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code SU3BKRKYT to authenticate.
Error from server (Forbidden): nodes is forbidden: User "f8b99861-f202-448e-9cb9-d147630c2786" cannot list resource "nodes" in API group "" at the cluster scope
```

# Cluster Info

```powershell
kubectl cluster-info
```


Now your production-ready AKS Cluster is ready to deploy Kubernetes applications. I will cover-up following areas in future tutorials. 

* Writing Kubernetes Manifest Step by Step 
* Azure DNS Zone configuration for AKS
* Kubernetes External DNS and Ingress Configuration 
* Ingress SSL configuration with Let's Encrypt.

### A Final Thought !!!

I hope you learn something new today. If you feel this tutorial worthwhile, Please visit my YouTube channel and hit the Subscribe and keep me posted. 

Good Luck & Stay Safe !!!











































