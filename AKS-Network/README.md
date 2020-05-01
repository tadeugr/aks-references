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