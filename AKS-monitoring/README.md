# Enable AKS monitoring add-on

## References

### Enable and review Kubernetes master node logs in Azure Kubernetes Service (AKS)

https://docs.microsoft.com/en-us/azure/aks/view-master-logs

### Create diagnostic setting to collect resource logs and metrics in Azure

https://docs.microsoft.com/en-us/azure/azure-monitor/platform/diagnostic-settings

# Telemetry and Healtchcheck

## References

### Monitor your Kubernetes cluster performance with Azure Monitor for containers

https://docs.microsoft.com/en-us/azure/azure-monitor/insights/container-insights-analyze

### Monitoring Azure Kubernetes Service (AKS) with Azure Monitor container health

https://azure.microsoft.com/en-us/blog/monitoring-azure-kubernetes-service-aks-with-azure-monitor-container-health-preview/

# Log debugging

## Examples

### Get Pods not running

```
KubePodInventory 
| where PodStatus != "Running" 
| where TimeGenerated > ago(24h) 
| top 100 by TimeGenerated desc
```

### Get Kubernetes events

```
KubeEvents 
| where TimeGenerated > ago(24h) 
| top 100 by TimeGenerated desc
```

### Check Kubernetes health

```
KubeHealth 
| where TimeGenerated > ago(24h) 
| top 100 by TimeGenerated desc
```

### Get nodes

```
KubeNodeInventory 
| where TimeGenerated > ago(24h) 
| top 100 by TimeGenerated desc
```

### Get Kubernetes services

```
KubeServices 
| limit 100
```

### Get container logs

```
let CONTAINER_NAME="my-container";
let ContainerIdList = KubePodInventory
| where ContainerName contains CONTAINER_NAME
| distinct ContainerID;
ContainerLog
| where ContainerID in (ContainerIdList)
| order by TimeGenerated desc
| render table
```

### Search for any "ERROR"

```
AzureDiagnostics
| where Category contains "kube-"
| where log_s contains "ERROR"
| top 100 by TimeGenerated desc
```

### Get user actions

```
AzureDiagnostics
| where Category contains "kube-"
| where log_s contains "john"
| top 100 by TimeGenerated desc
```