# Kubernetes Resource Management - Metrics Server & Memory Limits

This lab demonstrates:
1. **Deploying Metrics Server** for resource monitoring
2. **Memory requests and limits** – How they affect pod scheduling
3. **OOMKilled behavior** – When a pod exceeds its memory limit
4. **Pending pods** – When requests exceed node capacity

## 🎯 What you'll learn

- Install Metrics Server on a Kind cluster
- Use `kubectl top nodes` and `kubectl top pods`
- Configure memory `requests` and `limits`
- Understand why pods get `OOMKilled` or stay `Pending`

---

## 📋 Prerequisites

- Kubernetes cluster (Kind recommended)
- `kubectl` configured

---

## 🚀 Lab 1: Deploy Metrics Server

**Manifest:** [`metrics-server.yaml`](./metrics-server.yaml)

```bash
kubectl apply -f metrics-server.yaml
kubectl get pods -n kube-system -l k8s-app=metrics-server
```
Verify it works:
```bash
kubectl top nodes
kubectl top pods -n kube-system
```
### 🚀 Lab 2: Memory Requests and Limits

Create a namespace for the lab:
```bash
kubectl create namespace exmple-mem
```
Pod 1: Correct configuration

Manifest: memory-request-limit.yml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo
  namespace: exmple-mem
spec:
  containers:
  - name: memory-demo-ctr
    image: polinux/stress
    resources:
      requests:
        memory: "100Mi"
      limits:
        memory: "200Mi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
```
```bash
kubectl apply -f memory-request-limit.yml
kubectl get pods -n exmple-mem
```
Result: ✅ Running (150M fits within 200Mi limit)
Pod 2: Exceeds memory limit (OOMKilled)

Manifest: memory-demo2.yml
```yaml
# Same as above but with:
# requests: 50Mi, limits: 100Mi
# args: ["--vm-bytes", "250M"]
```
```bash
kubectl apply -f memory-demo2.yml
kubectl get pods -n exmple-mem
```
Result: ❌ OOMKilled (250M exceeds 100Mi limit)
Pod 3: Impossible request (Pending)

Manifest: memory-demo3.yml
```yaml
# requests: "1000Gi", limits: "1000Gi"
```
```bash
kubectl apply -f memory-demo3.yml
kubectl describe pod memory-demo-3 -n exmple-mem
```
Result: ⏳ Pending (no node has 1000Gi memory)
### 📊 Resource Comparison
Pod	Request	Limit	Stress	Status	Reason
memory-demo	100Mi	200Mi	150M	✅ Running	Stress fits within limit
memory-demo-2	50Mi	100Mi	250M	❌ OOMKilled	Stress exceeds limit
memory-demo-3	1000Gi	1000Gi	150M	⏳ Pending	Request impossible
### 📈 Verify Resource Usage
```bash
# Node-level metrics
kubectl top nodes

# Pod-level metrics
kubectl top pods -n exmple-mem
```
### 🧹 Clean Up
```bash
kubectl delete namespace exmple-mem
kubectl delete -f metrics-server.yaml
```
### 🔑 Key Takeaways
| Concept | Lesson |
|---------|--------|
| `requests` | Minimum resources reserved for the pod |
| `limits` | Maximum resources the pod can use |
| `OOMKilled` | Pod tried to exceed its memory limit |
| `Pending` | No node can satisfy the resource request |
| `Metrics Server` | Required for `kubectl top` and HPA |
