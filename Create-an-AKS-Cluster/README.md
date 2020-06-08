# Before you begin

## References

### What you can change after creating a cluster

#### az aks update

https://docs.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-update

#### az aks nodepool add

https://docs.microsoft.com/en-us/cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-add

#### az aks nodepool delete

https://docs.microsoft.com/en-us/cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-delete

#### az aks nodepool upgrade

https://docs.microsoft.com/en-us/cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-upgrade

#### az aks upgrade

https://docs.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-upgrade

#### az aks rotate-certs

https://docs.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-rotate-certs

#### az aks enable-addons

https://docs.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-enable-addons

#### az aks disable-addons

https://docs.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-disable-addons


# Using Azure Portal

## References

### Quickstart: Deploy an Azure Kubernetes Service (AKS) cluster using the Azure portal

https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal

# Using Azure CLI

## References

### Quickstart: Deploy an Azure Kubernetes Service cluster using the Azure CLI

https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough

### az aks create

https://docs.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-create

## Examples

```
az ad sp create-for-rbac \
  --skip-assignment \
  -n "REPLACE-WITH-YOUR-SP-NAME"
```

Ouput example.

```
{
  "appId": "b2a1cae2-4ad1-4cfe-8143-f10ba8b0580d",
  "displayName": "sp-aks",
  "name": "http://sp-aks",
  "password": "b9e253ef-ed80-44bf-ba51-5631f1e4caf6",
  "tenant": "cb9f0be6-95b6-46bb-9cd5-a67e5d5500a8"
}
```

Create a Resource Group.

```
az group create \
  --location REPLACE-WITH-YOUR-REGION \
  --name REPLACE-WITH-YOUR-AKS-RG-NAME \
  --subscription REPLACE-WITH-YOUR-SUBSCRIPTION \
  --tags 'ENV=DEV' 'SRV=MULTI-TAG-EXAMPLE'
```

### Create a cluster with kubenet and Standard LB

```
az aks create \
  --location REPLACE-WITH-YOUR-REGION \
  --subscription REPLACE-WITH-YOUR-SUBSCRIPTION \
  --resource-group REPLACE-WITH-YOUR-RG-NAME \
  --name REPLACE-WITH-YOUR-AKS-NAME \
  --no-ssh-key \
  --service-principal REPLACE-WITH-YOUR-SP-ID \
  --client-secret REPLACE-WITH-YOUR-SP-PASSWORD \
  --network-plugin kubenet \
  --load-balancer-sku standard \
  --outbound-type loadBalancer
```

### Create a cluster using an existing VNET/Subnet and SSH RSA key

Get the Subnet ID.

```
NET_RG="MC_REPLACE-WITH-YOUR-AKS-NODE-GROUP"
VNET_NAME="REPLACE-WITH-YOUR-VNET-NAME"
SUBNET_NAME="REPLACE-WITH-YOUR-SUBNET-NAME"
VNET_ID=$(az network vnet show --resource-group $NET_RG --name $VNET_NAME --query id -o tsv)
SUBNET_ID=$(az network vnet subnet show --resource-group $NET_RG --vnet-name $VNET_NAME --name $SUBNET_NAME --query id -o tsv)
```

Generate a new RSA key, use the file name `my-aks-ssh-key` for example.

```
ssh-keygen -t rsa -b 4096
```

```
az aks create \
  --name REPLACE-WITH-YOUR-AKS-NAME \
  --resource-group REPLACE-WITH-YOUR-AKS-RG-NAME \
  --subscription REPLACE-WITH-YOUR-SUBSCRIPTION \
  --dns-name-prefix REPLACE-WITH-YOUR-AKS-NAME \
  --dns-service-ip 10.0.0.10 \
  --docker-bridge-address 172.17.0.1/16 \
  --kubernetes-version 1.15.7 \
  --location REPLACE-WITH-YOUR-REGION \
  --network-plugin kubenet \
  --node-osdisk-size 500 \
  --node-vm-size Standard_F8s \
  --pod-cidr 10.244.0.0/16 \
  --service-cidr 10.0.0.0/16 \
  --load-balancer-sku standard \
  --enable-cluster-autoscaler \
  --node-count 2 \
  --min-count 2 \
  --max-count 3 \
  --vnet-subnet-id $SUBNET_ID \
  --ssh-key-value ./my-aks-ssh-key.pub \
  --tags 'ENV=DEV' 'SRV=TAGS-WILL-BE-PROPAGATED-TO-AKS-NODE-RG'
```

# [ HANDS-ON ] Using Azure CLI / ACR / K8S Dashboard

## Create ACR

Go to Azure Portal and create an ACR.

Import image.

```
az acr import  -n PASTE-YOUR-ACR-NAME-HERE --source docker.io/library/nginx:1.19.0 --image nginx:1.19.0
```

## Create an UID

```
RESOURCES_UID=$(uuidgen | cut -c1-8)
echo "RESOURCES_UID: $RESOURCES_UID" 
```

## AKS Network

```
VNET_NAME="vnet-aks-$RESOURCES_UID"
VNET_RG="rg-vnet-aks-$RESOURCES_UID"
VNET_REGION="eastus"
VNET_SUBSCRIPTION="DEV"
```

### Create the VNET RG

```
az group create \
  --name $VNET_RG \
  --location $VNET_REGION \
  --subscription $VNET_SUBSCRIPTION
```

### Create the VNET and Subnet

```
az network vnet create \
  --name $VNET_NAME \
  --resource-group $VNET_RG \
  --address-prefixes 192.0.0.0/8 \
  --subnet-name subnet-aks-$RESOURCES_UID \
  --subnet-prefixes 192.10.0.0/16 \
  --location $VNET_REGION \
  --subscription $VNET_SUBSCRIPTION
```

## Create AKS cluster

### Get subnet ID

```
VNET_NAME="vnet-aks-$RESOURCES_UID"
VNET_RG="rg-vnet-aks-$RESOURCES_UID"
SUBNET_NAME="subnet-aks-$RESOURCES_UID"
VNET_ID=$(az network vnet show --resource-group $VNET_RG --name $VNET_NAME --query id -o tsv)
SUBNET_ID=$(az network vnet subnet show --resource-group $VNET_RG --vnet-name $VNET_NAME --name $SUBNET_NAME --query id -o tsv)
```

### AKS env vars

```
AKS_NAME="aks-cluster-$RESOURCES_UID"
AKS_RG="rg-aks-cluster-$RESOURCES_UID"
AKS_REGION="eastus"
AKS_SUBSCRIPTION="DEV"
```

### AKS RG

```
az group create \
  --location $AKS_REGION \
  --name $AKS_RG \
  --subscription $AKS_SUBSCRIPTION
```

### Create SP

```
az ad sp create-for-rbac \
  --skip-assignment \
  -n "sp-aks"
```

### Add AKS SP Owner of the VNET RG


```
az role assignment create \
  --role "Owner" \
  --assignee "..." \
  --resource-group $VNET_RG
```

### Create AKS

```
az aks create \                     
  --location $AKS_REGION \
  --subscription $AKS_SUBSCRIPTION \
  --resource-group $AKS_RG \
  --name $AKS_NAME \
  --ssh-key-value $HOME/.ssh/id_rsa.pub \
  --service-principal "PASTE-YOUR-SP-ID-HERE" \
  --client-secret "PASTE-YOUR-SP-PASSWORD-HERE" \
  --network-plugin kubenet \
  --load-balancer-sku standard \
  --outbound-type loadBalancer \
  --vnet-subnet-id $SUBNET_ID \
  --pod-cidr 10.244.0.0/16 \
  --service-cidr 10.0.0.0/16 \
  --dns-service-ip 10.0.0.10 \
  --docker-bridge-address 172.17.0.1/16 \
  --node-vm-size Standard_B2s \
  --enable-cluster-autoscaler \
  --max-count 5 \
  --min-count 2 \
  --node-count 2 \
  --attach-acr "PASTE-YOUR-ARC-NAME-HERE" \
  --tags 'ENV=DEV' 'SRV=EXAMPLE'
```

Get kubeconfig.

```
FILE="./$AKS_NAME.kubeconfig"

az aks get-credentials \
  -n $AKS_NAME \
  -g $AKS_RG \
  --subscription $AKS_SUBSCRIPTION \
  --admin \
  --file $FILE
  
export KUBECONFIG=$FILE
```

## Deploy an app

Create a namifest file.

```
vim my-app.yaml
```

With the following content.

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - image: PASTE-YOUR-ACR-SERVER-HERE/IMAGE-REPO:TAG
        name: my-app
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: my-app
  type: LoadBalancer
```

Apply the manifest.

```
kubectl apply -f my-app.yaml
```

## K8s Dashboard

Go to: https://github.com/kubernetes/dashboard

And get the latest version.

### Dashboard permissions

You have two options:

1. Use your kubeconfig file;
2. Create a SA and use its token.

### RBAC

```
cat > ./dashboard-rbac.yml <<EOF
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dashboard-admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: dashboard-admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: dashboard-admin-user
  namespace: kubernetes-dashboard

EOF
```

Apply the manifest.

```
kubectl apply -f ./dashboard-rbac.yml
```

### Get the token

```
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secrets |grep dashboard-admin-user-token | awk '{print $1}')
```

### Access the dashboard

```
kubectl proxy
```

Access: http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

### Expose the dashboard to the Internet (please don't)

Create a manifest.

```
vim expose.yaml
```

With the following content

```
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-internet
  namespace: kubernetes-dashboard
spec:
  ports:
    - port: 443
      targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
  type: LoadBalancer
```

Apply the manifest.

```
kubectl apply -f expose.yaml -n kubernetes-dashboard
```