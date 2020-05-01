# Generate self-signed certificates

## References

### Request signing process

https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#request-signing-process

### A Practical Approach to Understanding Kubernetes Authentication

https://thenewstack.io/a-practical-approach-to-understanding-kubernetes-authentication/

## Examples

Generate a Private Key.

```
openssl genrsa -out bob.key 4096
```

Generate a Certificate Signing Request (CSR).

```
openssl req -new -key bob.key -out bob.csr -subj "/CN=bob/O=engineers"
```

Base64 encode the CSR.

```
cat bob.csr | base64 | tr -d '\n'
```

Create a `CertificateSigningRequest` manifest file.

```
vi signing-request.yaml
```

With the following content.

```
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: bob-csr
spec:
  groups:
  - system:authenticated
 
  request: PASTE-THE-CSR-BASE64-HERE
  
  usages:
  - digital signature
  - key encipherment
  - client auth
```

Create the CSR on Kubernentes.

```
kubectl create -f signing-request.yaml
```

Approve the CSR.

```
kubectl certificate approve bob-csr
```

Get the Signed Certificate (CRT).

```
kubectl get csr bob-csr -o jsonpath='{.status.certificate}' | base64 --decode > bob.crt
```

# Create kubeconfig files for users

## Examples

Create a kubeconfig file for a user using self-signed certificates, editing an orignal kubeconfig file.

Get an original kubeconfig file.

```
az aks get-credentials \
  --name YOUR-AKS-NAME
  --resource-group YOUR-AKS-RG \
  --file ./template.kubeconfig
  --admin
```

Get the The CRT base64 encoded.

```
cat bob.crt | base64 | tr -d '\n'
```

Get the The Key base64 encoded.

```
cat bob.key | base64 | tr -d '\n'
```

Edit the kubeconfig file.

```
vi ./template.kubeconfig
```

Keep the `.clusters.cluster.certificate-authority-data` of the original file.

Keep the `.clusters.cluster.server` of the original file.

Change `.contexts.context.user` and `.users.name`. They have to match.

Paste the "CRT BASE64" in the `.users.user.client-certificate-data`.

Paste the "KEY BASE64" in the `.users.user.client-key-data`.

The kubeconfig file bellow is just na example.

```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JS...
    server: https://aks-lb-sta-aks-network-fbca3e-563769af.hcp.brazilsouth.azmk8s.io:443
  name: aks-lb-standard
contexts:
- context:
    cluster: aks-lb-standard
    user: bob_aks-network_aks-lb-standard
  name: aks-lb-standard-admin
current-context: aks-lb-standard-admin
kind: Config
preferences: {}
users:
- name: bob_aks-network_aks-lb-standard
  user:
    client-certificate-data: PASTE THE "CRT BASE64" HERE
    client-key-data: PASTE THE "KEY BASE64" HERE
```

# RBAC for Kubernetes Users and Groups

## Examples

## RBAC for Users

`.subjects.apiGroup.name` has to match certificate's Common Name (`CN`).

```
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: crb-admin-usr-bob
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: bob
```

## RBAC for Groups

`.subjects.apiGroup.name` has to match certificate's organization (`O`).

```
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: crb-admin-usr-bob
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: engineers
```