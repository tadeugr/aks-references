# Create the server application

## References

### Create the server application

https://docs.microsoft.com/en-us/azure/aks/azure-ad-integration#create-the-server-application

### Create Azure AD server component (CLI)

https://docs.microsoft.com/en-us/azure/aks/azure-ad-integration-cli#create-azure-ad-server-component

# Create the client application

## References 

### Create the client application

https://docs.microsoft.com/en-us/azure/aks/azure-ad-integration#create-the-client-application

### Create Azure AD client component (CLI)

https://docs.microsoft.com/en-us/azure/aks/azure-ad-integration-cli#create-azure-ad-client-component

# Integrate AKS with Azure Active Directory

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

Generate kubeconfig file.

```
az aks get-credentials \
  --subscription REPLACE-WITH-YOUR-SUBSCRIPTION \
  --resource-group REPLACE-WITH-YOUR-AKS-RG \
  --name REPLACE-WITH-YOUR-AKS-NAME \
  --file ./REPLACE-WITH-YOUR-FILE-NAME.kubeconfig
```

Example of kubeconfig generated.

```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiJDLWNGWELDQ0FURS0tLS0tCk1JSUV5VENDQXJHZ0F3SUJBZ0lRTDl6RGlHc3dZczF1WFBLQjRJeThIekFOQmdrcWhraUc5dzBCQVFzRkFEQU4KTVFzd0NRWURWUVFERXdKallUQWdGdzB5TURBeU1Ua3hNekEzTXpsYUdBOHlNRFV3TURJeE1URXpNVGN6T1ZvdwpEVEVMTUFrR0ExVUVBeE1DWTJFd2dnSWlNQTBHQ1NxR1NJYjNEUUVCQVFVQUE0SUNEd0F3Z2dJS0FvSUNBUURHCmtlOTVRQ0h6Z3hOSVlzdnc1N3NqVDZLOHVuZVNUcFp2M0E3OFlPMmkxeUFicFlkWEZNRDc5UXppTytkRXIrYjMKcFlBZTBIOStMMmVxaUQzY3IzdCs4M0J1M0pDUzFIU3VsTEhJNFdOeUFMWTc0REtLeUVnNzZnK090cjNUTHJnUwpsalV0My93YnBmRXF2ZC9wT0p0ejBLYTNYcTRGUEl3Q3N3MDV0anRMUjBvUlhXNGF1UUd1cDBZZDFMbVJ1QzQvCnJ0b2NuOTQ1NjNQSk5MamJ4LzhzQm9DNGR1Z2lCdWZ2aDNvck85K1NyLzZtdTRxeWc2MlRFWWNBVm9uNkl3bmcKbkxRVUtQUjVCbk9mSUNmZ0VFUHcwOWVEWE9Nd1VEdVFQVlVVNmoyUzQxK251Y3g1ci9qWkxXM3RBZmtVblY0MwpMdjlRYjJZSWJVbTBoNTIxV3ZRQ1lWSFZWczdtdjd0QUl2WDBISWQ1YTlqN1hxeUpla1NsOXNoQ3o5WWlaeHVRCmVJUm41TTZ3TWQrVGJ5TGZXdlZpVlFhSlhkWUdDb2xVd0VpcitBdjF1OExoMEt0WmE3NkF1MUxVbHRROXNYYWwKUlY0N1gwWUxWdE1USmdzSHJaWmZrK2FqNVpFT1Nnb1pEeWxYZGNDUGE5UDRLRUo2b2xRcmtTNjlKOUkzSVpKdwpEcHZJZURrckttRVlFNkdDeU1ybjdrdHZ5cmZJNmk0NmlzdzY5dlZWaUhNRExjOHRMSlo0aUttLzhDZzBuRFVXCnRxZlIxeW1NWDVFaVVsRGhucXFlQ3p0TmxyOGh5eG9OdXA5NG9nTXVIcGxKZXNYZzNhUjBVNm1EejBCbWpvOVIKQmcrdnZhOXdiS0pMbDdNcVRadDJPNWlyWkdsSHY4SEhjMmpBeGNiWEl3SURBUUFCb3lNd0lUQU9CZ05WSFE4QgpBZjhFQkFNQ0FxUXdEd1lEVlIwVEFRSC9CQVV3QXdFQi96QU5CZ2txaGtpRzl3MEJBUXNGQUFPQ0FnRUFqMFBiClhsbUx2U3FFL2tKM0hKd2liazhUK0J4Snl1bjlhblZYd3NxaDdrb25pV0ZxaW1BR2pHNzRuZUZzc0JMWGt5V2oKd09TMXFtTnk3cHl2SDZ1TnlSTjNyWlo0VWZZWnA5TXJKVmU0c0RmUm1jU2pSTWN0ZEJ6K2ZOSzVUT1NvVUxKegpZMFFpeFF1TllMR1NPY1hGN24yNmVMY051YmZUSWZrVEQ4Q3VhakxjL2l2UER2VXdaUVlEWTcvN2VncUdZTzZ6Cko4ZzNpMDJBRm5OazRNU2xiQWxHeW0yM2hWbVNaakJBK2Z3cURhaVRpYW5sTzVNNVFHMlNlajhMa0FyaWNBTEgKQy9wMERmUFJSdzBKemgrWkY5OFJsZjcxYWp3RVR4NytLNXJ0aGVHaFV1bzhvclQxbUMxakU3RUk1RnlHQ3Y5cApFRlJwQWZPZnJhVW0wYXJ2cklKYk5xM0ZSMi8vZ2o4WWhpL0JGRDk2Y2tFa1JLRFFaUnFkbjBMTlE0Tm81VHhVClo1eEl6Q3YyRjhNMkw5NCtIbnV4L2c0bHV6RFZzTmU1ekxZSXhteGhJOStFM0NnM0FRQmhWdDlLbXZtamkwM0EKNTlTY1RzTStNWnJxaVoranovSVREM0xzdFAvVi9yM0habStBcHV3UXd4RHhHbmNzeFdmN3doaVcyMXYyYlBqWAo5OFRqdW0vallhR3pOekRnaGZlSFpOdzk1WlNOSTE5elEvbHVJVGRrc2FRK0FvSWVna1ZoeEN1SFh6bFdRSjR6CmhQYkZCTGFGZzhHSnN4Z1BvQ29FNDBOeVl1N2I2eVJLTUJiSU5RWHhVZ0JOakpyUmluS21iOU5KK3ZmaS85bksKTGRhd1gwSk8wZ21tdVdrRzIrM200M3BNM3F1SU83UGQ4NDV4VDA4PQotLS0tLUJHDWDBKWENSDUZJQ0FURS0tLS0tCg==
    server: https://YOUR-CLUSTER-NAME-8bf7aad5.hcp.eastus.azmk8s.io:443
  name: YOUR-CLUSTER-NAME
contexts:
- context:
    cluster: YOUR-CLUSTER-NAME
    user: clusterUser_YOUR-RG_YOUR-CLUSTER-NAME
  name: YOUR-CLUSTER-NAME
current-context: YOUR-CLUSTER-NAME
kind: Config
preferences: {}
users:
- name: clusterUser_YOUR-RG_YOUR-CLUSTER-NAME
  user:
    auth-provider:
      config:
        apiserver-id: YOUR-SERVER-APP-ID
        client-id: YOUR-CLIENT-APP-ID
        config-mode: '1'
        environment: AzurePublicCloud
        tenant-id: YOUR-TENANT-ID
      name: azure
```

Use the kubeconfig file created.

```
export KUBECONFIG=./REPLACE-WITH-YOUR-FILE-NAME.kubeconfig
```

# Kubernetes RBAC for AAD Users and Groups

## "AKS-ROOT" group

Create a group on AAD with all users that will have full admin access to Kubernetes.

```
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: crb-cluster-admin-grp-engineers
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  # REPLACE-WITH-YOUR-AAD-GROUP-NAME
  name: "REPLACE-WITH-YOUR-AAD-GROUP-OBJECT-ID"
```

## "AKS-READ-ONLY" group

Create a group on AAD with all users that will have read-only access to the entire Kubernetes cluster.

```
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: cr-adgrp-cluster-reader
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: crb-adgrp-cluster-reader
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cr-adgrp-cluster-reader
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  # REPLACE-WITH-YOUR-AAD-GROUP-NAME
  name: "REPLACE-WITH-YOUR-AAD-GROUP-OBJECT-ID"
```

## "NS-ADMIN" group

Create one group on AAD for each squad with users that will have full access to a specific namespace.

```
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: Role
metadata:
  name: r-REPLACE-WITH-YOUR-SQUAD-NAME-adgrp-ns-admin
  namespace: REPLACE-WITH-YOUR-SQUAD-NAME
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: rb-REPLACE-WITH-YOUR-SQUAD-NAME-adgrp-ns-admin
  namespace: REPLACE-WITH-YOUR-SQUAD-NAME
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: r-REPLACE-WITH-YOUR-SQUAD-NAME-adgrp-ns-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  # REPLACE-WITH-YOUR-AAD-GROUP-NAME
  name: "REPLACE-WITH-YOUR-AAD-GROUP-OBJECT-ID"
```