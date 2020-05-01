# Provisioners

## References

### Provisioner

https://kubernetes.io/docs/concepts/storage/storage-classes/#provisioner

# Dynamic Volumes

## References

### Dynamically create and use a persistent volume with Azure disks in Azure Kubernetes Service (AKS)

https://docs.microsoft.com/en-us/azure/aks/azure-disks-dynamic-pv

### Dynamically create and use a persistent volume with Azure Files in Azure Kubernetes Service (AKS)

https://docs.microsoft.com/en-us/azure/aks/azure-files-dynamic-pv

## Examples

### Create a dynamic volume using azure-disk SC

Create a PVC and azure will provision a disk and create the PV.

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: azure-managed-disk
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: managed-premium
  resources:
    requests:
      storage: 5Gi
```

Mount the volume in a Pod.

```
kind: Pod
apiVersion: v1
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: nginx
    volumeMounts:
    - mountPath: "/mnt/azure"
      name: volume
  volumes:
    - name: volume
      persistentVolumeClaim:
        claimName: azure-managed-disk
```

### Create a dynamic volume using azure-file SC

Create a PVC and azure will provision a Storage Class and create the PV.

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: azurefile
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: azurefile
  resources:
    requests:
      storage: 5Gi
```

Mount the volume in a Pod.

```
kind: Pod
apiVersion: v1
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: nginx
    volumeMounts:
    - mountPath: "/mnt/azure"
      name: volume
  volumes:
    - name: volume
      persistentVolumeClaim:
        claimName: azurefile
```

# Static Volumes

## References

### Manually create and use a volume with Azure disks in Azure Kubernetes Service (AKS)

https://docs.microsoft.com/en-us/azure/aks/azure-disk-volume

### Manually create and use a volume with Azure Files share in Azure Kubernetes Service (AKS)

https://docs.microsoft.com/en-us/azure/aks/azure-files-volume

## Examples

### Use a static Disk (azure-disk)

Get the disk URI.

```
az disk show \
  --name MY-DISK-NAME \
  --resource-group MY-DISK-RESOURCE-GROUP \
  --subscription MY-DISK-SUBSCRIPTION
```

Output example.

```
...
"id": "/subscriptions/7a87b86a-eecb-4e2a-8a94-2360e443dba3/resourceGroups/MY-DISK-RESOURCE-GROUP/providers/Microsoft.Compute/disks/MY-DISK-NAME",
...
```

Mount the volume in a Pod.

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - image: nginx:1.15.5
    name: mypod
    volumeMounts:
      - name: azure
        mountPath: /mnt/azure
  volumes:
      - name: azure
        azureDisk:
          kind: Managed
          diskName: REPLACE-WITH-YOUR-DISK-NAME
          diskURI: REPLACE-WITH-YOUR-DISK-URI
```

### Use a static Storage Account (azure-file)

Go to Azure Portal and get your Storage Account Name and Key.

Export these env vars.

```
STORAGE_ACCOUNT_NAME="sa20c11e0a"
STORAGE_KEY="Z2WjB+uBVksEiA+3A8pq19MYQfTcIZQzXaRowrW9Twe+1AW+MppRIxFw=="
```

Create a Kubernetes secret.

```
kubectl create secret generic azure-secret \
  --from-literal=azurestorageaccountname=$STORAGE_ACCOUNT_NAME \
  --from-literal=azurestorageaccountkey=$STORAGE_KEY
```

#### Mount a static Storage Account directly in the Pod

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - image: nginx:1.15.5
    name: mypod
    volumeMounts:
      - name: azure
        mountPath: /mnt/azure
  volumes:
  - name: azure
    azureFile:
      secretName: azure-secret
      shareName: REPLACE-WITH-YOUR-SHARE-NAME
      readOnly: false
```

#### Mount a static Storage Account using a PV and a PVC

Create a PV.

Notice the `my-static-file-link`. It is not a proper `StorageClass`, it is only a reference to be used in the PVC.

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-static-file-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteMany
  storageClassName: my-static-file-link
  azureFile:
    secretName: azure-secret
    shareName: REPLACE-WITH-YOUR-SHARE-NAME
    readOnly: false
  mountOptions:
  - dir_mode=0777
  - file_mode=0777
  - uid=1000
  - gid=1000
  - mfsymlinks
  - nobrl
```

Create a PVC.

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-static-file-pvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: my-static-file-link
  resources:
    requests:
      storage: 5Gi
```

Mount it in a Pod.

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - image: nginx:1.15.5
    name: mypod
    volumeMounts:
      - name: azure
        mountPath: /mnt/azure
  volumes:
  - name: azure
    persistentVolumeClaim:
      claimName: my-static-file-pvc
```
