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