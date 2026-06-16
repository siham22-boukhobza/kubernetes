# Kubernetes Day 14 - Certificates & RBAC

This lab demonstrates **Certificate Signing Requests (CSR)** and **RBAC** (Role-Based Access Control) in Kubernetes.

## 🎯 What you'll learn

- Generate and approve CSR for user authentication
- Configure kubeconfig with client certificates
- Create ClusterRoles and ClusterRoleBindings
- Test user permissions with `kubectl auth can-i`

---

## 📋 Prerequisites

- Kubernetes cluster (Kind recommended)
- `kubectl` configured
- `openssl` installed (for certificate generation)

---

## 🚀 Part 1: Certificate Signing Request (CSR)

### 1. Generate private key

```bash
openssl genrsa -out akram.key 2048
```
### 2. Generate CSR
```bash
openssl req -new -key akram.key -out akram.csr -subj "/CN=akram"
```
### 3. Encode CSR in base64
```bash
cat akram.csr | base64 -w 0
```
### 4. Create CSR manifest (csr.yml)
```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: akram
spec:
  signerName: kubernetes.io/kube-apiserver-client
  request: <base64-encoded-csr>
  expirationSeconds: 86400
  usages:
  - client auth
```
### 5. Apply and approve CSR
```bash
kubectl apply -f csr.yml
kubectl get csr
kubectl certificate approve akram
```
### 6. Extract certificate
```bash
kubectl get csr akram -o jsonpath='{.status.certificate}' | base64 -d > akram.crt
```
---

### 🚀 Part 2: Configure kubeconfig
### 1. Set user credentials
```bash
kubectl config set-credentials akram --client-key=akram.key --client-certificate=akram.crt --embed-certs=true
```
### 2. Create context
```bash
kubectl config set-context akram --cluster=kind-cluster-three --user=akram
```
### 3. Switch to context
```bash
kubectl config use-context akram
```
### 4. Test authentication
```bash
kubectl auth whoami
kubectl get pods
```
---

### 🚀 Part 3: RBAC - ClusterRole & ClusterRoleBinding
### 1. Generate ClusterRole manifest
```bash
kubectl create clusterrole node-reader --verb=get,watch,list --resource=nodes --dry-run=client -o yaml > clusterrole.yml
```
### 2. Generate ClusterRoleBinding manifest
```bash
kubectl create clusterrolebinding reader-nodes --clusterrole=node-reader --user=akram --dry-run=client -o yaml > clusterrolebinding.yml
```
### 3. Apply RBAC configuration
```bash
kubectl apply -f clusterrole.yml
kubectl apply -f clusterrolebinding.yml
```
### 4. Test permissions
```bash
kubectl auth can-i get nodes
kubectl auth can-i delete nodes
kubectl get nodes
```
---

### 📊 Verification commands
```bash
# List CSR
kubectl get csr

# Describe CSR
kubectl describe csr akram

# List ClusterRoles
kubectl get clusterrole | grep node-reader

# Describe ClusterRole
kubectl describe clusterrole node-reader

# List ClusterRoleBindings
kubectl get clusterrolebinding | grep reader-nodes

# Check user permissions
kubectl auth can-i --list --as=akram
```
---

### 🧹 Clean Up
```bash
# Delete CSR
kubectl delete csr akram

# Delete ClusterRole and Binding
kubectl delete clusterrole node-reader
kubectl delete clusterrolebinding reader-nodes

# Delete context
kubectl config delete-context akram

# Delete user credentials
kubectl config unset users.akram
```
---

### 🔑 Key Takeaways
|Concept|Lesson|
|-------|------|
|`CSR`|Allows secure certificate issuance without sharing private keys|
|`kubectl certificate approve`|Approves certificate requests|
|`set-credentials`|Adds user credentials to kubeconfig|
|`set-context`|Creates context linking cluster and user|
|`ClusterRole`|Defines permissions at cluster level|
|`ClusterRoleBinding`|Binds ClusterRole to a user|
|`kubectl auth can-i`|Tests user permissions|
