# Kubernetes Scheduling - Taints, Tolerations & Node Selectors

This repository demonstrates **Kubernetes scheduling constraints** using:
1. **Taints & Tolerations** - Repel pods from nodes
2. **Node Selectors** - Attract pods to specific nodes

## 🎯 What you'll learn

- Add/remove taints on nodes
- Pods without toleration cannot schedule on tainted nodes
- Node labels and `nodeSelector` for pod placement
- Scheduling behavior when constraints are (not) met

---

## 📋 Prerequisites

- Kubernetes cluster with at least 2 worker nodes
- `kubectl` configured

---

## 🚀 Lab 1: Taints & Tolerations

### Step 1: Add taint to worker nodes

```bash
kubectl taint node cluster-three-worker gpu=true:NoSchedule
kubectl taint node cluster-three-worker2 gpu=true:NoSchedule
```
### Step 2: Create a pod WITHOUT toleration

```bash
kubectl run nginx --image=nginx
kubectl get pods -w
```
Result: Pod stays in `Pending` state
```bash
NAME    READY   STATUS    RESTARTS   AGE
nginx   0/1     Pending   0          10s
```
### Step 3: Check why pod is pending
```bash
kubectl describe pod nginx
```

Expected message:
```text
0/3 nodes are available: 3 node(s) had untolerated taint(s).
```

### Step 4: Remove taint from one node
```bash
kubectl taint node cluster-three-worker2 gpu=true:NoSchedule-
```
### Step 5: Pod schedules on untainted node
```bash
kubectl get pods -o wide
```
Result:
```text
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE
nginx   1/1     Running   0          30s   10.244.2.2   cluster-three-worker2
```
### Step 6: Create a pod WITH toleration

Manifest: pod-taint-toleration.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
  labels:
    run: redis
spec:
  containers:
  - name: redis
    image: redis
  tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
```
```bash
kubectl apply -f pod-taint-toleration.yaml
kubectl get pods -o wide
```
Result: Pod schedules on tainted node (has toleration)
```text
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE
redis   1/1     Running   0          5s    10.244.1.3   cluster-three-worker
```
## 🚀 Lab 2: Node Selectors
### Step 1: Check existing node labels
```bash
kubectl get nodes --show-labels
```
### Step 2: Create pod with nodeSelector

Manifest: pod-nodeselector.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-new
  labels:
    run: nginx-new
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    gpu: "false"
```
```bash
kubectl apply -f pod-nodeselector.yaml
kubectl get pods -w
```
Result: Pod stays in Pending (no node has label gpu=false)
### Step 3: Add label to a node
```bash
kubectl label node cluster-three-worker2 gpu=false
```
### Step 4: Pod schedules on labeled node
```bash
kubectl get pods -o wide
```
Result:
```text
NAME        READY   STATUS    RESTARTS   AGE   IP           NODE
nginx-new   1/1     Running   0          10s   10.244.2.3   cluster-three-worker2
```
### 📊 Comparison: `Taints` vs `Node Selectors`
| Concept | Purpose | Effect |
|---|---|---|
| `Taint` | Repel pods from node | Pods without toleration cannot schedule |
| `Toleration` | Allow pod on tainted node | Pod can schedule on tainted node |

🧹 Clean Up
```bash
# Delete pods
kubectl delete pod nginx redis nginx-new
```
# Remove taints
```bash
kubectl taint node cluster-three-worker gpu=true:NoSchedule-
kubectl taint node cluster-three-worker2 gpu=true:NoSchedule-
```
# Remove labels
```bash
kubectl label node cluster-three-worker2 gpu-
```
🔑 Key Takeaways
`Concept`	        `Lesson`
`Taints`	Nodes  can be configured to repel pods
`Tolerations`	Pods can be allowed onto tainted nodes
`Node Selector`	Pods can be restricted to specific nodes
`Node Labels`	Nodes can be categorized for scheduling
`Scheduling`	Pods only run on nodes satisfying ALL constraints

---

### 📄 **Manifest files**

**`pod-taint-toleration.yaml`**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
  labels:
    run: redis
spec:
  containers:
  - name: redis
    image: redis
  tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
```
pod-nodeselector.yaml
```yaml

apiVersion: v1
kind: Pod
metadata:
  name: nginx-new
  labels:
    run: nginx-new
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    gpu: "false"
```

