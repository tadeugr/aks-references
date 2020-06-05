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