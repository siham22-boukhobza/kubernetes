# Kubernetes Init Containers - Hands-On Lab

This repository demonstrates **Kubernetes Init Containers** with two progressive examples:
1. **Single init container** - Waiting for one service
2. **Multiple init containers** - Waiting for two services (sequential)

## 🎯 What you'll learn

- Init containers block main container startup
- Service discovery with `nslookup`
- Sequential execution of multiple init containers
- Pod lifecycle: `Init:0/1` → `Init:1/1` → `PodInitializing` → `Running`

---

## 📋 Example 1: Single Init Container

**Manifest:** [`single-init-pod.yaml`](./single-init-pod.yaml)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo "app running!" && sleep 3600']
  initContainers:
  - name: init-app
    image: busybox:1.28
    command: ['sh', '-c']
    args: ['until nslookup my-service.default.svc.cluster.local; do echo waiting...; sleep 2; done']
```
🚀 Run it
```bash
# 1. Create pod (init container waits)
kubectl apply -f single-init-pod.yaml

# 2. Watch status (stuck in Init:0/1)
kubectl get pods -w

### 3. Create service to satisfy init container
kubectl create service clusterip my-service --port 80

# 4. Pod transitions: Init:0/1 → Init:1/1 → Running
kubectl get pods

# 5. Verify
kubectl logs app-pod
```
📋 Example 2: Multiple Init Containers (Sequential)

Manifest: multiple-init-pod.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo "app running!" && sleep 3600']
  initContainers:
  - name: init-app
    image: busybox:1.28
    command: ['sh', '-c']
    args: ['until nslookup my-service.default.svc.cluster.local; do echo waiting for my-service...; sleep 2; done']
  - name: db-app
    image: busybox:1.28
    command: ['sh', '-c']
    args: ['until nslookup my-db.default.svc.cluster.local; do echo waiting for my-db...; sleep 2; done']
```
🚀 Run it
```bash
# 1. Create pod (first init container waits)
kubectl apply -f multiple-init-pod.yaml

# 2. Create first service (satisfies init-app)
kubectl create service clusterip my-service --port 80

# 3. First init completes, second init runs (waiting for my-db)
kubectl logs app-pod -c db-app --follow

# 4. Create second service (satisfies db-app)
kubectl create service clusterip my-db --port 3306

# 5. Pod transitions to Running
kubectl get pods
```
📊 Status Timeline
Status	               `Meaning`
`Init:0/2`	        First init container running
`Init:1/2`	        First done, second running
`PodInitializing`	Both done, main starting
`Running`	        Main container running
🧹 Clean Up
```bash
# Delete pods
kubectl delete pod app-pod 

# Delete services
kubectl delete service my-service my-db 
```

🔑 Key Takeaways
### Concept Lesson
`Init containers` Run before main container, must complete successfully
`Multiple init containers` Run sequentially (one after another)
Services Only need to exist for DNS resolution (no pods required)
Pod lifecycle	Init:X/Y shows progress through init containers
```text
---

### 📄 **Manifest files**

**single-init-pod.yaml**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
  labels:
    name: init-demo
spec:
  containers:
  - name: app-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo "the application is running!" && sleep 3600']
  initContainers:
  - name: init-app
    image: busybox:1.28
    command: ['sh', '-c']
    args: ['until nslookup my-service.default.svc.cluster.local; do echo waiting for my-service; sleep 2; done']
```
multiple-init-pod.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
  labels:
    name: init-demo
spec:
  containers:
  - name: app-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo "the application is running!" && sleep 3600']
  initContainers:
  - name: init-app
    image: busybox:1.28
    command: ['sh', '-c']
    args: ['until nslookup my-service.default.svc.cluster.local; do echo waiting for my-service; sleep 2; done']
  - name: db-app
    image: busybox:1.28
    command: ['sh', '-c']
    args: ['until nslookup my-db.default.svc.cluster.local; do echo waiting for my-db; sleep 2; done']
```
```
