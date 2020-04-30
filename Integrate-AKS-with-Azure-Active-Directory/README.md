# Create the server application

## References

### Create the server application

https://docs.microsoft.com/en-us/azure/aks/azure-ad-integration#create-the-server-application

# Create the client application

## References 

### Create the client application

https://docs.microsoft.com/en-us/azure/aks/azure-ad-integration#create-the-client-application

# Creante an AKS cluster integrated with AAD

## References

### Deploy the AKS cluster

https://docs.microsoft.com/en-us/azure/aks/azure-ad-integration#deploy-the-aks-cluster

### az aks create

https://docs.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-create

## Examples

Create an AKS cluster integrated with AAD.

```
az aks create \
  --location REPLACE-WITH-YOUR-REGION \
  --subscription REPLACE-WITH-YOUR-SUBSCRIPTION \
  --resource-group REPLACE-WITH-YOUR-AKS-RG \
  --name REPLACE-WITH-YOUR-AKS-NAME \
  --no-ssh-key \
  --service-principal REPLACE-WITH-YOUR-AKS-SP \
  --client-secret REPLACE-WITH-YOUR-AKS-SP-PASSWORD \
  --aad-server-app-id REPLACE-WITH-YOUR-SERVER-APP-ID \
  --aad-server-app-secret "REPLACE-WITH-YOUR-SERVER-APP-SECRET" \
  --aad-client-app-id REPLACE-WITH-YOUR-CLIENT-APP-ID \
  --aad-tenant-id REPLACE-WITH-YOUR-TENANT-ID \
  --network-plugin kubenet \
  --load-balancer-sku standard \
  --outbound-type loadBalancer
```

# Update AAD integration credentials

## References

### az aks update-credentials

https://docs.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-update-credentials

## Examples

```
az aks update-credentials \
  --subscription REPLACE-WITH-YOUR-SUBSCRIPTION \
  --resource-group REPLACE-WITH-YOUR-AKS-RG \
  --name REPLACE-WITH-YOUR-AKS-NAME \
  --aad-server-app-id REPLACE-WITH-YOUR-NEW-SERVER-APP-ID \
  --aad-server-app-secret "REPLACE-WITH-YOUR-NEW-SERVER-APP-SECRET" \
  --aad-client-app-id REPLACE-WITH-YOUR-NEW-CLIENT-APP-ID \
  --aad-tenant-id REPLACE-WITH-YOUR-NEW-TENANT-ID \
  --reset-aad
```

# Generate kubeconfig (get-credentials)

## References

### az aks get-credentials

https://docs.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials

## Examples

```
az aks get-credentials \
  --subscription REPLACE-WITH-YOUR-SUBSCRIPTION \
  --resource-group REPLACE-WITH-YOUR-AKS-RG \
  --name REPLACE-WITH-YOUR-AKS-NAME \
  --file ./REPLACE-WITH-YOUR-FILE-NAME.kubeconfig
```