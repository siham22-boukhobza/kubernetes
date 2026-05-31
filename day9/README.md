# Kubernetes Node Affinity - Hard vs Soft Scheduling

This repository demonstrates **Kubernetes Node Affinity** using two types:
1. **`requiredDuringSchedulingIgnoredDuringExecution`** (Hard affinity) - Pod MUST match the label
2. **`preferredDuringSchedulingIgnoredDuringExecution`** (Soft affinity) - Pod PREFERS but doesn't require the label

## 🎯 What you'll learn

- Add labels to nodes
- Hard affinity: pod stays `Pending` if label doesn't exist
- Soft affinity: pod schedules elsewhere if label doesn't exist
- Difference between `requiredDuringScheduling` and `preferredDuringScheduling`

---

## 📋 Prerequisites

- Kubernetes cluster with at least 2 worker nodes
- `kubectl` configured

---

## 🚀 Lab 1: Hard Affinity (Required)

**Manifest:** [`affinity.yml`](./affinity.yml)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
  labels:
    run: redis
spec:
  containers:
  - name: redis-container
    image: redis
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
 ```
 Step 1: Check existing node labels
```bash
kubectl get nodes --show-labels
```
Step 2: Deploy the pod (if no node has disktype=ssd)
```bash
kubectl apply -f affinity.yml
kubectl get pods -w
```
Result: Pod stays in Pending state
```text
NAME    READY   STATUS    RESTARTS   AGE
redis   0/1     Pending   0          10s
```
Step 3: Check why pod is pending
```bash
kubectl describe pod redis
```
Expected message:
```text
0/3 nodes are available: 3 node(s) didn't match Pod's node affinity/selector.
```
Step 4: Add label to a node
```bash
kubectl label node cluster-three-worker disktype=ssd
```
Step 5: Pod schedules immediately
```bash
kubectl get pods -o wide
```
Result:
```text
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE
redis   1/1     Running   0          5s    10.244.1.3   cluster-three-worker
```
### 🚀 Lab 2: Soft Affinity (Preferred)

Manifest: affinity2.yml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis-new
  labels:
    run: redis
spec:
  containers:
  - name: redis-container
    image: redis
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        preference:
          matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
```
Step 1: Deploy the pod (without disktype=ssd label on all nodes)
```bash
kubectl apply -f affinity2.yml
kubectl get pods -o wide
```
Result: Pod schedules immediately on any available node (preference is ignored if not met)
```text
NAME        READY   STATUS    RESTARTS   AGE   IP           NODE
redis-new   1/1     Running   0          5s    10.244.2.4   cluster-three-worker2
```
Step 2: Add preferred label to a node
```bash
kubectl label node cluster-three-worker2 disktype=ssd
```
Step 3: Delete and recreate the pod
```bash
kubectl delete pod redis-new
kubectl apply -f affinity2.yml
kubectl get pods -o wide
```
Result: Pod now schedules on the node with disktype=ssd
```text
NAME        READY   STATUS    RESTARTS   AGE   IP           NODE
redis-new   1/1     Running   0          5s    10.244.2.4   cluster-three-worker2
```
### 📊 Comparison: Hard vs Soft Affinity
| Type | Keyword | Behavior | Use case |
|------|---------|----------|----------|
| Hard | `requiredDuringScheduling` | Pod MUST match label, else stays Pending | Workloads that require specific hardware (SSD, GPU) |
| Soft | `preferredDuringScheduling` | Pod PREFERS label, but schedules elsewhere if needed | Workloads that perform better with certain resources |
### 🧹 Clean Up
```bash
# Delete pods
kubectl delete pod redis redis-new

# Remove labels
kubectl label node cluster-three-worker disktype-
kubectl label node cluster-three-worker2 disktype-
```
🔑 Key Takeaways
Concept	Lesson
`Hard Affinity`	Pod is blocked until label exists
`Soft Affinity`	Pod schedules immediately using best effort
`Node Labels`	Essential for directing pod placement
`matchExpressions`	Flexible matching (In, NotIn, Exists, DoesNotExist)
