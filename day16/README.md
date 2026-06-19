# Kubernetes Day 16 - Network Policies with Calico

This lab demonstrates **Network Policies** in Kubernetes using **Calico** as the CNI.

## 🎯 What you'll learn

- Deploy a multi-service application (frontend, backend, database)
- Install Calico CNI on Kind
- Apply Network Policies to control traffic flow
- Test connectivity between services

---

## 📋 Prerequisites

- `kind` installed
- `kubectl` configured

---

## 🚀 Cluster Setup

### 1. Create Kind cluster with Calico

```bash
kind create cluster --config cluster.yml --name calico-cluster
```
cluster.yml:
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
networking:
  disableDefaultCNI: true
  podSubnet: 192.168.0.0/16
```
---

### 2. Install Calico
```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.1/manifests/calico.yaml
```
---

### 3. Verify Calico is running
```bash
kubectl get pods -n kube-system -l k8s-app=calico-node
```
---

### 🚀 Application Deployment
### 4. Deploy the application
```bash
kubectl apply -f app.yml

Resources created:

    Pods: frontend, backend, mysql

    Services: frontend, backend, db
```
---

### 5. Verify deployment
```bash
kubectl get pods
kubectl get svc
```
---

### 🚀 Network Policies
### 6. Apply Network Policies
```bash
kubectl apply -f network-policies.yml
```
Policies applied:

    ✅ allow-frontend-to-backend

    ❌ deny-frontend-to-db

    ✅ allow-backend-to-db

### 🧪 Connectivity Tests
Test 1: frontend → backend (✅ Allowed)
```bash
kubectl exec -it frontend -- curl backend:80
```
Expected: Nginx welcome page
Test 2: frontend → db (❌ Blocked)
```bash
kubectl exec -it frontend -- telnet db 3306
```
Expected: Timeout or connection refused
Test 3: backend → db (✅ Allowed)
```bash
kubectl exec -it backend -- telnet db 3306
```
Expected: MySQL connection established
### 📊 Network Policies Summary
| Policy | Source | Destination | Action |
|---|---|---|---|
| `allow-frontend-to-backend` | frontend | backend:80 | ✅ Allow |
| `deny-frontend-to-db` | frontend | db:3306 | ❌ Deny |
| `allow-backend-to-db` | backend | db:3306 | ✅ Allow |
---

## 🧹 Clean Up
```bash
# Delete application
kubectl delete -f app.yml

# Delete network policies
kubectl delete -f network-policies.yml

# Delete cluster
kind delete cluster --name calico-cluster
```
---

### 🔑 Key Takeaways
|Concept|Lesson|
|-------|------|
|Calico	CNI|that supports Network Policies|
|NetworkPolicy|Controls traffic between pods|
|podSelector|Selects pods to apply policy|
|Ingress|Controls incoming traffic|

