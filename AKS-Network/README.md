# AKS network architecture

## References

### Network concepts for applications in Azure Kubernetes Service (AKS)

https://docs.microsoft.com/en-us/azure/aks/concepts-network

### Create a private Azure Kubernetes Service cluster

https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/aks/private-clusters.md

# Kubenet (basic) vs Azure CNI (advanced)

## References

### Compare network models

https://docs.microsoft.com/en-us/azure/aks/concepts-network#compare-network-models

# Basic Load Balancer vs Standard Load Balancer

## References

### Deep dive – Azure Load Balancer

https://msandbu.org/deep-dive-azure-load-balancer/

# AKS Egress

## References

### Use a Standard SKU load balancer in Azure Kubernetes Service (AKS)

https://docs.microsoft.com/en-us/azure/aks/load-balancer-standard

### Control egress traffic for cluster nodes in Azure Kubernetes Service (AKS)

https://docs.microsoft.com/en-us/azure/aks/limit-egress-traffic

### Outbound connections in Azure

https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-outbound-connections

### Understanding SNAT and PAT

https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-outbound-connections#snat

### Configure outbound ports and idle timeout

https://docs.microsoft.com/en-us/azure/aks/load-balancer-standard#configure-outbound-ports-and-idle-timeout

### Control egress traffic for cluster nodes in Azure Kubernetes Service (AKS)

https://docs.microsoft.com/en-us/azure/aks/limit-egress-traffic

## Examples

### Create an internal LB

Notice the `azure-load-balancer-internal` annotation. 

```
kind: Service
apiVersion: v1
metadata:
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-internal: "true"
  name: my-service
  namespace: default
spec:
  ports:
  - protocol: TCP
    port: 60000
  type: LoadBalancer
```

### Create an external LB

```
kind: Service
apiVersion: v1
metadata:
  name: my-custom-egress
  namespace: default
spec:
  ports:
  - protocol: TCP
    port: 60000
  type: LoadBalancer
```

# AKS Ingress

## References

### HTTP application routing

https://docs.microsoft.com/en-us/azure/aks/http-application-routing

## Examples

Deploy an example app.

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: party-clippy
spec:
  template:
    metadata:
      labels:
        app: party-clippy
    spec:
      containers:
      - image: r.j3ss.co/party-clippy
        name: party-clippy
        tty: true
        command: ["party-clippy"]
        ports:
        - containerPort: 8080
```

Create a service.

```
apiVersion: v1
kind: Service
metadata:
  name: party-clippy
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 8080
  selector:
    app: party-clippy
  type: ClusterIP
```

Get your DNS zone domain and replace in the file bellow.

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: party-clippy
  annotations:
    kubernetes.io/ingress.class: addon-http-application-routing
spec:
  rules:
  - host: party-clippy.REPLACE-WITH-YOUR-DNS-DOMAIN.aksapp.io
    http:
      paths:
      - backend:
          serviceName: party-clippy
          servicePort: 80
        path: /
```

Then access in your browser.

http://party-clippy.REPLACE-WITH-YOUR-DNS-DOMAIN.aksapp.io

# [ HANDS-ON ] Basic External Load Balancer with dynamic outbound IP


## Export env vars

```
AKS_NAME="REPLACE-WITH-YOUR-CLUSTER-NAME"
AKS_RG="REPLACE-WITH-YOUR-CLUSTER-RESOURCE-GROUP"
SUBSCRIPTION="REPLACE-WITH-YOUR-CLUSTER-SUBSCRIPTION"
REGION="REPLACE-WITH-YOUR-CLUSTER-REGION"
```

## Create RG

```
az group create \
  --location $REGION \
  --name $AKS_RG \
  --subscription $SUBSCRIPTION
```

## Create SP

```
az ad sp create-for-rbac \
  --skip-assignment \
  -n "sp-aks"
```

## Create AKS

```
az aks create \
  --location $REGION \
  --subscription $SUBSCRIPTION \
  --resource-group $AKS_RG \
  --name $AKS_NAME \
  --ssh-key-value $HOME/.ssh/id_rsa.pub \
  --service-principal "REPLACE-WITH-YOUR-SP-ID" \
  --client-secret "REPLACE-WITH-YOUR-SP-PASSWORD" \
  --network-plugin kubenet \
  --load-balancer-sku basic \
  --outbound-type loadBalancer \
  --node-vm-size Standard_B2s \
  --node-count 1 \
  --tags 'ENV=DEV' 'SRV=EXAMPLE'
```

Get kubeconfig.

```
FILE="./$AKS_NAME.kubeconfig"

az aks get-credentials \
  -n $AKS_NAME \
  -g $AKS_NAME_RG \
  --subscription $SUBSCRIPTION \
  --admin \
  --file $FILE

export KUBECONFIG=$FILE
```

## Deploy Pod

```
kubectl run -it --rm --generator=run-pod/v1 busybox --image=busybox -- sh
```

## Create VM

Create a Virtual Machine to be the AKS destination.

### Export env vars

```
VM_NAME="REPLACE-WITH-YOUR-VM-NAME"
VM_RG="REPLACE-WITH-YOUR-CLUSTER-VM-GROUP"
SUBSCRIPTION="REPLACE-WITH-YOUR-VM-SUBSCRIPTION"
REGION="REPLACE-WITH-YOUR-VM-REGION"
```

### Create RG

```
az group create \
  --location $REGION \
  --name $VM_RG \
  --subscription $SUBSCRIPTION
```

### Create VM

```
az vm create \
  --location $REGION \
  --subscription $SUBSCRIPTION \
  --resource-group $VM_RG \
  --name $VM_NAME \
  --ssh-key-values $HOME/.ssh/id_rsa.pub \
  --admin-username devops \
  --image UbuntuLTS \
  --nsg-rule SSH \
  --public-ip-address-allocation static \
  --public-ip-sku Basic \
  --size Standard_A2_v2
```

### Use netcat to start a process

SSH to the VM and run:

```
nc -l 8080
```

### Use tcpdump to inspect packets 

SSH to the VM and run:

```
tcpdump -n -i eth0 port 8080
```

Then, from the AKS Pod, run: 

```
telnet IP.OF.OUR.VM 8080
```

# [ HANDS-ON ] Basic External Load Balancer with static outbound IP


## Export env vars

```
AKS_NAME="REPLACE-WITH-YOUR-CLUSTER-NAME"
AKS_RG="REPLACE-WITH-YOUR-CLUSTER-RESOURCE-GROUP"
SUBSCRIPTION="REPLACE-WITH-YOUR-CLUSTER-SUBSCRIPTION"
REGION="REPLACE-WITH-YOUR-CLUSTER-REGION"
```

## Create RG

```
az group create \
  --location $REGION \
  --name $AKS_RG \
  --subscription $SUBSCRIPTION
```

## Create SP

```
az ad sp create-for-rbac \
  --skip-assignment \
  -n "sp-aks"
```

## Create AKS

```
az aks create \
  --location $REGION \
  --subscription $SUBSCRIPTION \
  --resource-group $AKS_RG \
  --name $AKS_NAME \
  --ssh-key-value $HOME/.ssh/id_rsa.pub \
  --service-principal "REPLACE-WITH-YOUR-SP-ID" \
  --client-secret "REPLACE-WITH-YOUR-SP-PASSWORD" \
  --network-plugin kubenet \
  --load-balancer-sku basic \
  --outbound-type loadBalancer \
  --node-vm-size Standard_B2s \
  --node-count 1 \
  --tags 'ENV=DEV' 'SRV=EXAMPLE'
```

Get kubeconfig.

```
FILE="./$AKS_NAME.kubeconfig"

az aks get-credentials \
  -n $AKS_NAME \
  -g $AKS_NAME_RG \
  --subscription $SUBSCRIPTION \
  --admin \
  --file $FILE

export KUBECONFIG=$FILE
```

## Create service

Apply the following manifest.

```
kind: Service
apiVersion: v1
metadata:
  name: my-custom-egress
spec:
  ports:
  - protocol: TCP
    port: 60000
  type: LoadBalancer
```

## Deploy Pod

```
kubectl run -it --rm --generator=run-pod/v1 busybox --image=busybox -- sh
```

## Create VM

Create a Virtual Machine to be the AKS destination.

### Export env vars

```
VM_NAME="REPLACE-WITH-YOUR-VM-NAME"
VM_RG="REPLACE-WITH-YOUR-CLUSTER-VM-GROUP"
SUBSCRIPTION="REPLACE-WITH-YOUR-VM-SUBSCRIPTION"
REGION="REPLACE-WITH-YOUR-VM-REGION"
```

### Create RG

```
az group create \
  --location $REGION \
  --name $VM_RG \
  --subscription $SUBSCRIPTION
```

### Create VM

```
az vm create \
  --location $REGION \
  --subscription $SUBSCRIPTION \
  --resource-group $VM_RG \
  --name $VM_NAME \
  --ssh-key-values $HOME/.ssh/id_rsa.pub \
  --admin-username devops \
  --image UbuntuLTS \
  --nsg-rule SSH \
  --public-ip-address-allocation static \
  --public-ip-sku Basic \
  --size Standard_A2_v2
```

### Use netcat to start a process

SSH to the VM and run:

```
nc -l 8080
```

### Use tcpdump to inspect packets 

SSH to the VM and run:

```
tcpdump -n -i eth0 port 8080
```

Then, from the AKS Pod, run: 

```
telnet IP.OF.OUR.VM 8080
```

# [ HANDS-ON ] Standard External Load Balancer


## Export env vars

```
AKS_NAME="REPLACE-WITH-YOUR-CLUSTER-NAME"
AKS_RG="REPLACE-WITH-YOUR-CLUSTER-RESOURCE-GROUP"
SUBSCRIPTION="REPLACE-WITH-YOUR-CLUSTER-SUBSCRIPTION"
REGION="REPLACE-WITH-YOUR-CLUSTER-REGION"
```

## Create RG

```
az group create \
  --location $REGION \
  --name $AKS_RG \
  --subscription $SUBSCRIPTION
```

## Create SP

```
az ad sp create-for-rbac \
  --skip-assignment \
  -n "sp-aks"
```

## Create AKS

```
az aks create \
  --location $REGION \
  --subscription $SUBSCRIPTION \
  --resource-group $AKS_RG \
  --name $AKS_NAME \
  --ssh-key-value $HOME/.ssh/id_rsa.pub \
  --service-principal "REPLACE-WITH-YOUR-SP-ID" \
  --client-secret "REPLACE-WITH-YOUR-SP-PASSWORD" \
  --network-plugin kubenet \
  --load-balancer-sku standard \
  --outbound-type loadBalancer \
  --node-vm-size Standard_B2s \
  --node-count 1 \
  --tags 'ENV=DEV' 'SRV=EXAMPLE'
```

Get kubeconfig.

```
FILE="./$AKS_NAME.kubeconfig"

az aks get-credentials \
  -n $AKS_NAME \
  -g $AKS_NAME_RG \
  --subscription $SUBSCRIPTION \
  --admin \
  --file $FILE

export KUBECONFIG=$FILE
```

## Deploy Pod

```
kubectl run -it --rm --generator=run-pod/v1 busybox --image=busybox -- sh
```

## Create VM

Create a Virtual Machine to be the AKS destination.

### Export env vars

```
VM_NAME="REPLACE-WITH-YOUR-VM-NAME"
VM_RG="REPLACE-WITH-YOUR-CLUSTER-VM-GROUP"
SUBSCRIPTION="REPLACE-WITH-YOUR-VM-SUBSCRIPTION"
REGION="REPLACE-WITH-YOUR-VM-REGION"
```

### Create RG

```
az group create \
  --location $REGION \
  --name $VM_RG \
  --subscription $SUBSCRIPTION
```

### Create VM

```
az vm create \
  --location $REGION \
  --subscription $SUBSCRIPTION \
  --resource-group $VM_RG \
  --name $VM_NAME \
  --ssh-key-values $HOME/.ssh/id_rsa.pub \
  --admin-username devops \
  --image UbuntuLTS \
  --nsg-rule SSH \
  --public-ip-address-allocation static \
  --public-ip-sku Basic \
  --size Standard_A2_v2
```

### Use netcat to start a process

SSH to the VM and run:

```
nc -l 8080
```

### Use tcpdump to inspect packets 

SSH to the VM and run:

```
tcpdump -n -i eth0 port 8080
```

Then, from the AKS Pod, run: 

```
telnet IP.OF.OUR.VM 8080
```

# [ HANDS-ON ] Internal network egress

## Create an UID

```
RESOURCES_UID=$(uuidgen | cut -c1-8)
echo "RESOURCES_UID: $RESOURCES_UID" 
```

## AKS Network

```
VNET_NAME="vnet-web-$RESOURCES_UID"
VNET_RG="rg-vnet-web-$RESOURCES_UID"
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
  --subnet-name subnet-aks-kubenet-$RESOURCES_UID \
  --subnet-prefixes 192.10.0.0/16 \
  --location $VNET_REGION \
  --subscription $VNET_SUBSCRIPTION
```

### Create another Subnet (aks-azure-cni)

```
az network vnet subnet create \
  --name subnet-aks-azure-cni-$RESOURCES_UID \
  --resource-group $VNET_RG \
  --vnet-name $VNET_NAME \
  --address-prefixes 192.20.0.0/16
```

### Create another Subnet (VM)

```
az network vnet subnet create \
  --name subnet-web-$RESOURCES_UID \
  --resource-group $VNET_RG \
  --vnet-name $VNET_NAME \
  --address-prefixes 192.30.0.0/16
```


## VM Network

```
VNET_NAME="vnet-database-$RESOURCES_UID"
VNET_RG="rg-vnet-database-$RESOURCES_UID"
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
  --address-prefixes 240.0.0.0/16 \
  --subnet-name subnet-db-$RESOURCES_UID \
  --subnet-prefixes 240.0.0.0/18 \
  --location $VNET_REGION \
  --subscription $VNET_SUBSCRIPTION
```

### Create the Peering

Go to Azure Portal and create the Peering.

## Create Virtual Machines

### VM "Web"

#### Get subnet ID

```
VNET_NAME="vnet-web-$RESOURCES_UID"
VNET_RG="rg-vnet-web-$RESOURCES_UID"
SUBNET_NAME="subnet-web-$RESOURCES_UID"
VNET_ID=$(az network vnet show --resource-group $VNET_RG --name $VNET_NAME --query id -o tsv)
SUBNET_ID=$(az network vnet subnet show --resource-group $VNET_RG --vnet-name $VNET_NAME --name $SUBNET_NAME --query id -o tsv)
```

#### Env vars

```
VM_NAME="vm-web-$RESOURCES_UID"
VM_RG="rg-vm-web-$RESOURCES_UID"
VM_REGION="eastus"
VM_SUBSCRIPTION="DEV"
```

#### Create RG

```
az group create \
  --location $VM_REGION \
  --name $VM_RG \
  --subscription $VM_SUBSCRIPTION
```

### Create VM

```
az vm create \
  --location $VM_REGION \
  --subscription $VM_SUBSCRIPTION \
  --resource-group $VM_RG \
  --name $VM_NAME \
  --ssh-key-values $HOME/.ssh/id_rsa.pub \
  --admin-username devops \
  --image UbuntuLTS \
  --nsg-rule SSH \
  --subnet $SUBNET_ID \
  --public-ip-address-allocation static \
  --public-ip-sku Basic \
  --size Standard_A2_v2
```

### VM "Database"

#### Get subnet ID

```
VNET_NAME="vnet-database-$RESOURCES_UID"
VNET_RG="rg-vnet-database-$RESOURCES_UID"
SUBNET_NAME="subnet-db-$RESOURCES_UID"
VNET_ID=$(az network vnet show --resource-group $VNET_RG --name $VNET_NAME --query id -o tsv)
SUBNET_ID=$(az network vnet subnet show --resource-group $VNET_RG --vnet-name $VNET_NAME --name $SUBNET_NAME --query id -o tsv)
```

#### Env vars

```
VM_NAME="vm-database-$RESOURCES_UID"
VM_RG="rg-vm-database-$RESOURCES_UID"
VM_REGION="eastus"
VM_SUBSCRIPTION="DEV"
```

#### Create RG

```
az group create \
  --location $VM_REGION \
  --name $VM_RG \
  --subscription $VM_SUBSCRIPTION
```

### Create VM

```
az vm create \
  --location $VM_REGION \
  --subscription $VM_SUBSCRIPTION \
  --resource-group $VM_RG \
  --name $VM_NAME \
  --ssh-key-values $HOME/.ssh/id_rsa.pub \
  --admin-username devops \
  --image UbuntuLTS \
  --nsg-rule SSH \
  --subnet $SUBNET_ID \
  --public-ip-address-allocation static \
  --public-ip-sku Basic \
  --size Standard_A2_v2
```

## Create AKS cluster with Kubenet

### Get subnet ID

```
VNET_NAME="vnet-web-$RESOURCES_UID"
VNET_RG="rg-vnet-web-$RESOURCES_UID"
SUBNET_NAME="subnet-aks-kubenet-$RESOURCES_UID"
VNET_ID=$(az network vnet show --resource-group $VNET_RG --name $VNET_NAME --query id -o tsv)
SUBNET_ID=$(az network vnet subnet show --resource-group $VNET_RG --vnet-name $VNET_NAME --name $SUBNET_NAME --query id -o tsv)
```

### Create AKS

#### AKS env vars

```
AKS_NAME="aks-kubenet-$RESOURCES_UID"
AKS_RG="rg-aks-kubenet-$RESOURCES_UID"
AKS_REGION="eastus"
AKS_SUBSCRIPTION="DEV"
```

#### AKS RG

```
az group create \
  --location $AKS_REGION \
  --name $AKS_RG \
  --subscription $AKS_SUBSCRIPTION
```

#### Create SP

```
az ad sp create-for-rbac \
  --skip-assignment \
  -n "sp-aks"
```

#### Add AKS SP Owner of the VNET RG


```
az role assignment create \
  --role "Owner" \
  --assignee "a0c9b27a-fe3a-4353-9565-5a32a95cfa90" \
  --resource-group $VNET_RG
```

#### Create AKS

```
az aks create \
  --location $AKS_REGION \
  --subscription $AKS_SUBSCRIPTION \
  --resource-group $AKS_RG \
  --name $AKS_NAME \
  --ssh-key-value $HOME/.ssh/id_rsa.pub \
  --service-principal "a0c9b27a-fe3a-4353-9565-5a32a95cfa90" \
  --client-secret "FBeNGB7UkhO_w~SKq9fijZfI_7YJkqlh1c" \
  --network-plugin kubenet \
  --load-balancer-sku standard \
  --outbound-type loadBalancer \
  --vnet-subnet-id $SUBNET_ID \
  --node-vm-size Standard_B2s \
  --node-count 2 \
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
```

## Create AKS cluster with Azure CNI

### Get subnet ID

```
VNET_NAME="vnet-web-$RESOURCES_UID"
VNET_RG="rg-vnet-web-$RESOURCES_UID"
SUBNET_NAME="subnet-aks-azure-cni-$RESOURCES_UID"
VNET_ID=$(az network vnet show --resource-group $VNET_RG --name $VNET_NAME --query id -o tsv)
SUBNET_ID=$(az network vnet subnet show --resource-group $VNET_RG --vnet-name $VNET_NAME --name $SUBNET_NAME --query id -o tsv)
```

### Create AKS

#### AKS env vars

```
AKS_NAME="aks-azure-cni-$RESOURCES_UID"
AKS_RG="aks-azure-cni-$RESOURCES_UID"
AKS_REGION="eastus"
AKS_SUBSCRIPTION="DEV"
```

#### AKS RG

```
az group create \
  --location $AKS_REGION \
  --name $AKS_RG \
  --subscription $AKS_SUBSCRIPTION
```

#### Create SP

```
az ad sp create-for-rbac \
  --skip-assignment \
  -n "sp-aks"
```

#### Add AKS SP Owner of the VNET RG


```
az role assignment create \
  --role "Owner" \
  --assignee "a0c9b27a-fe3a-4353-9565-5a32a95cfa90" \
  --resource-group $VNET_RG
```

#### Create AKS

```
az aks create \
  --location $AKS_REGION \
  --subscription $AKS_SUBSCRIPTION \
  --resource-group $AKS_RG \
  --name $AKS_NAME \
  --ssh-key-value $HOME/.ssh/id_rsa.pub \
  --service-principal "a0c9b27a-fe3a-4353-9565-5a32a95cfa90" \
  --client-secret "FBeNGB7UkhO_w~SKq9fijZfI_7YJkqlh1c" \
  --network-plugin azure \
  --load-balancer-sku standard \
  --outbound-type loadBalancer \
  --vnet-subnet-id $SUBNET_ID \
  --node-vm-size Standard_B2s \
  --node-count 2 \
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
```

## Create a Daemonset

Create a Daemonset, than connecto the pods to run tests.

Apply the following manifets.

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ds-busybox
spec:
  selector:
    matchLabels:
      name: busybox
  template:
    metadata:
      labels:
        name: busybox
    spec:
      containers:
      - name: busybox
        image: busybox
        command: [ "/bin/sh", "-c", "--" ]
        args: [ "tail -f /dev/null" ]
```

# [ HANDS-ON ] External nginx-ingress / cert-manager (letsencrypt) / external-dns

# Buy a domain

Go to Azure Portal, App Service Domains and buy a new domain. Azure will create the domain and a DNS Zone for the domain.

## Create a SP

```
az ad sp create-for-rbac -n sp-external-dns
```

Copy the output with the SP credentials, you will use it later.


## Get the DNS Zone RG ID

On Azure Portal, go to your DNS Zone Resource Group, click on Properties and copy its ID.


## Get the DNS Zone ID

On Azure Portal, go to your DNS Zone, click on Properties and copy its ID.


## The SP must be "Reader" of the DNS Zone RG

```
az role assignment create \
  --role "Reader" \
  --assignee "PASTE-YOUR-SP-ID-HERE" \
  --scope "PASTE-YOUR-DNS-ZONE-RG-ID-HERE"
```

## The SP must be "Contributor" of the DNS Zone

```
az role assignment create \
  --role "Contributor" \
  --assignee "PASTE-YOUR-SP-ID-HERE" \
  --scope "PASTE-YOUR-DNS-ZONE-ID-HERE"
```

## Get the Tenant ID

```
az account show --query "tenantId"
```

## Get the Subscription ID

```
az account show --query "id"
```

## Get the RG of the DNS Zone

On Azure Portal, get the DNS Zone Resource Group name.

## Create credentials file

```
vim azure.json
```

Paste the following content.

```
{
  "tenantId": "PASTE-YOUR-TENANT-ID-HERE",
  "subscriptionId": "PASTE-YOUR-SUBSCRIPTION-ID-HERE",
  "resourceGroup": "PASTE-YOUR-DNS-ZONE-RG-NAME-HERE",
  "aadClientId": "PASTE-YOUR-SP-ID-HERE",
  "aadClientSecret": "PASTE-YOUR-SP-SECRET-HERE"
}
```

## Create a Kubernetes secret

```
kubectl create ns my-app

kubectl create secret generic azure-config-file --from-file=./azure.json -n my-project
```

## Install nginx ingress

```
kubectl create namespace ingress-external

helm repo add stable https://kubernetes-charts.storage.googleapis.com/

helm repo update

helm upgrade -i ingress-external stable/nginx-ingress \
    --namespace ingress-external \
    --set controller.publishService.enabled=true \
    --set controller.publishService.pathOverride="ingress-external/ingress-external-nginx-ingress-controller" \
    --set controller.replicaCount=2 \
    --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
    --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux
```

## Install external-dns

Go to: https://github.com/kubernetes-sigs/external-dns/releases

And get the Docker image tag of the latest release.

Create a manifest file.

```
vim external-dns.yaml
```

With the following content.

```
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: external-dns
  namespace: my-app
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: external-dns
rules:
- apiGroups: [""]
  resources: ["services","endpoints","pods"]
  verbs: ["get","watch","list"]
- apiGroups: ["extensions"] 
  resources: ["ingresses"] 
  verbs: ["get","watch","list"]
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["list"]
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: external-dns-viewer
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: external-dns
subjects:
- kind: ServiceAccount
  name: external-dns
  namespace: my-app
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: external-dns
  namespace: my-app
spec:
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: external-dns
  template:
    metadata:
      labels:
        app: external-dns
    spec:
      serviceAccountName: external-dns
      containers:
      - name: external-dns
        image: us.gcr.io/k8s-artifacts-prod/external-dns/external-dns:v0.7.2
        args:
        - --source=service
        - --source=ingress
        - --domain-filter=my-domain.com
        - --provider=azure
        - --azure-resource-group=REPLACE-WITH-YOUR-DNS-ZONE-RG
        volumeMounts:
        - name: azure-config-file
          mountPath: /etc/kubernetes
          readOnly: true
      volumes:
      - name: azure-config-file
        secret:
          secretName: azure-config-file
```

Apply the manifest.

```
kubectl -n my-app apply -f external-dns.yaml
```

## Install cert-manager

Go to: https://cert-manager.io/docs/installation/kubernetes/#steps

And see which is the latest version.

Install cert-manager.

```
kubectl create namespace cert-manager

helm repo add jetstack https://charts.jetstack.io

helm repo update

helm upgrade -i cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --version v0.15.1 \
  --set installCRDs=true
```
Get the pods.

```
kubectl get pods -n cert-manager
```

Create a manifest.

```
vim letsencrypt.yaml
```

With the following content.

```
apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
  name: letsencrypt
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: tadeugraz@outlook.com
    privateKeySecretRef:
      name: letsencrypt
    solvers:
    - http01:
        ingress:
          class: nginx
```

Apply the manifest

```
kubectl apply -f letsencrypt.yaml
```

Describe the cluster issuer.

```
kubectl describe clusterissuers letsencrypt
```

## Test it

Create a manifest.

```
vim app-my-app.yaml
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
      - image: r.j3ss.co/party-clippy
        name: my-app
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 8080
  selector:
    app: my-app
  type: ClusterIP

---
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: my-app
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: "letsencrypt"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  tls:
  - hosts:
    - app.my-domain.com
    secretName: tls-secret
  rules:
  - host: app.my-domain.com
    http:
      paths:
      - backend:
          serviceName: my-app
          servicePort: 80
        path: /(/|$)(.*)
```

Apply the manifest.

```
kubectl -n my-app apply -f app-my-app.yaml
```

Get the certificate.

```
kubectl get certificate -n my-app
```

## References


https://docs.microsoft.com/en-us/azure/aks/ingress-tls

https://github.com/kubernetes-sigs/external-dns/blob/master/docs/tutorials/azure.md

https://github.com/helm/charts/tree/master/stable/nginx-ingress

https://cert-manager.io/docs/installation/kubernetes/

# [ HANDS-ON ] Internal nginx-ingress / cert-manager (self-signed) / external-dns

## Create Link DNS Zone

Go to Azure Portal, open Private DNS zones and a new Private DNS zones.

Make sure your Private DNS zone is linked
to AKS VNET.

## Create a SP

```
az ad sp create-for-rbac -n sp-internal-dns
```

Copy the output with the SP credentials, you will use it later.


## Get the DNS Zone RG ID

On Azure Portal, go to your DNS Zone Resource Group, click on Properties and copy its ID.


# Get the DNS Zone ID

On Azure Portal, go to your DNS Zone, click on Properties and copy its ID.


## The SP must be "Reader" of the DNS Zone RG

```
az role assignment create \
  --role "Reader" \
  --assignee "PASTE-YOUR-SP-ID-HERE" \
  --scope "PASTE-YOUR-PRIVATE-DNS-ZONE-RG-ID-HERE"
```
 
## The SP must be "Private DNS Zone Contributor" of the DNS Zone

```
az role assignment create \
  --role "Private DNS Zone Contributor" \
  --assignee "PASTE-YOUR-SP-ID-HERE" \
  --scope "PASTE-YOUR-PRIVATE-DNS-ZONE-ID-HERE"
```

## Get the Tenant ID

```
az account show --query "tenantId"
```

## Get the Subscription ID

```
az account show --query "id"
```

## Get the RG of the DNS Zone

On Azure Portal, get the DNs Zone Resource Group name.

## Install nginx ingress

```
kubectl create namespace ingress-internal

helm repo add stable https://kubernetes-charts.storage.googleapis.com/

helm repo update

helm upgrade -i ingress-internal stable/nginx-ingress \
    --namespace ingress-internal \
    --set controller.publishService.enabled=true \
    --set controller.publishService.pathOverride="ingress-internal/ingress-internal-nginx-ingress-controller" \
    --set controller.service.annotations."service\.beta\.kubernetes\.io\/azure-load-balancer-internal"=true \
    --set controller.replicaCount=2 \
    --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
    --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux
```

## Install external-dns

Go to: https://github.com/kubernetes-sigs/external-dns/releases

And get the Docker image tag of the latest release.

Create a namespce.

```
kubectl create ns internal-app
```

Create manifest file.

```
vim external-dns.yaml
```

With the following content.

```
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: external-dns
  namespace: internal-app
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: external-dns
rules:
- apiGroups: [""]
  resources: ["services","endpoints","pods"]
  verbs: ["get","watch","list"]
- apiGroups: ["extensions"] 
  resources: ["ingresses"] 
  verbs: ["get","watch","list"]
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["list"]
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: external-dns-viewer
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: external-dns
subjects:
- kind: ServiceAccount
  name: external-dns
  namespace: internal-app
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: external-dns
  namespace: internal-app
spec:
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: external-dns
  template:
    metadata:
      labels:
        app: external-dns
    spec:
      serviceAccountName: external-dns
      containers:
      - name: external-dns
        image: us.gcr.io/k8s-artifacts-prod/external-dns/external-dns:v0.7.2
        args:
        - --source=service
        - --source=ingress
        - --domain-filter=internal.com
        - --provider=azure-private-dns
        - --azure-resource-group=rg-private-dns
        - --azure-subscription-id=c498bf57-778b-4495-81f4-79da718e9c56
        env:
        - name: AZURE_TENANT_ID
          value: "7743c76f-8f1e-4711-ab3a-6c4ad905bcd0"
        - name: AZURE_CLIENT_ID
          value: "84856f5f-3553-4633-bd1f-817fc4c693ee"
        - name: AZURE_CLIENT_SECRET
          value: "zw~QjVcatl5oc5PMfoKKsq2919MWm01e7d"
```

Apply the manifest.

```
kubectl -n internal-app apply -f external-dns.yaml
```

## Install cert-manager

Go to: https://cert-manager.io/docs/installation/kubernetes/#steps

And see which is the latest version.

Install cert-manager

```
kubectl create namespace cert-manager

helm repo add jetstack https://charts.jetstack.io

helm repo update

helm upgrade -i cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --version v0.15.1 \
  --set installCRDs=true
```

List the pods.

```
kubectl get pods -n cert-manager
```

Create a manifest file.

```
vim selfsigned-issuer.yaml
```

With the following content.

```
---
apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
  name: selfsigned-issuer
spec:
  selfSigned: {}
```

Apply the manifest.

```
kubectl apply -f selfsigned-issuer.yaml
```

Describe the Cluster Issuer.

```
kubectl describe clusterissuers selfsigned-issuer
```

## Test it

Create a manifest file.

```
vim app-internal-app.yaml
```

With the following content.

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: internal-app
spec:
  selector:
    matchLabels:
      app: internal-app
  template:
    metadata:
      labels:
        app: internal-app
    spec:
      containers:
      - image: r.j3ss.co/party-clippy
        name: internal-app
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: internal-app
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 8080
  selector:
    app: internal-app
  type: ClusterIP

---
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: internal-app
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: "selfsigned-issuer"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  tls:
  - hosts:
    - app.internal.com
    secretName: tls-secret
  rules:
  - host: app.internal.com
    http:
      paths:
      - backend:
          serviceName: internal-app
          servicePort: 80
        path: /(/|$)(.*)
```

Apply the manifest.

```
kubectl -n internal-app apply -f app-internal-app.yaml
```

Get the certificate.

```
kubectl get certificate -n internal-app
```

## References


https://github.com/kubernetes-sigs/external-dns/blob/master/docs/tutorials/azure-private-dns.md

https://cert-manager.io/docs/configuration/selfsigned/
