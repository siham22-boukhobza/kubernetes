# Kubernetes Namespace Hands-On Lab

This guide demonstrates how to use **Kubernetes Namespaces** to isolate resources, enable cross-namespace communication, and manage services.

## 🎯 Objectives

- Create and manage namespaces
- Deploy pods in different namespaces
- Test pod-to-pod connectivity
- Use Services with FQDN (Fully Qualified Domain Name)
- Scale deployments
- Clean up resources

---

## 📋 Prerequisites

- Kubernetes cluster (Kind, Minikube, or cloud-based)
- `kubectl` configured

---

## 🚀 Tasks & Answers

### 1. Create two namespaces

```bash
kubectl create namespace ns1
kubectl create namespace ns2
```

### 2. Create a deployment in each namespace
```bash
kubectl create deploy deploy-ns1 --image=nginx -n ns1
kubectl create deploy deploy-ns2 --image=nginx -n ns2
```

### 3. Get the IP address of each pod
```bash
kubectl get pods -n ns1 -o wide
kubectl get pods -n ns2 -o wide
```

### 4. Test pod-to-pod connection

Exec into pod in ns1 and curl the pod IP from ns2:
```bash
kubectl exec -it <pod-name-ns1> -n ns1 -- sh
# curl <pod-ip-ns2>
```

Exec into pod in ns2 and curl the pod IP from ns1:
```bash
kubectl exec -it <pod-name-ns2> -n ns2 -- sh
# curl <pod-ip-ns1>
```

✅ Result: Communication should succeed (network policy defaults to allow).
### 5. Scale deployments
```bash
kubectl scale --replicas=3 deploy/deploy-ns1 -n ns1
kubectl scale --replicas=3 deploy/deploy-ns2 -n ns2
```
### 6. Create services to expose deployments
```bash

kubectl expose deploy deploy-ns1 --name=svc-ns1 --port=80 -n ns1
kubectl expose deploy deploy-ns2 --name=svc-ns2 --port=80 -n ns2
```
### 7. Get service ClusterIPs
```bash
kubectl get svc -n ns1
kubectl get svc -n ns2
```
### 8. Test cross-namespace service connectivity

Exec into pod in ns1 and curl service ClusterIP from ns2:
```bash
kubectl exec -it <pod-name-ns1> -n ns1 -- sh
# curl <cluster-ip-svc-ns2>
```
✅ Result: Should work.
### 9. Test DNS resolution of service
```bash
kubectl exec -it <pod-name-ns1> -n ns1 -- sh
# curl svc-ns1
# curl svc-ns2
```

❌ curl svc-ns2 fails — the short name only resolves within its own namespace.
### 10. Use `FQDN` (Fully Qualified Domain Name)

Find the cluster domain:
```bash
kubectl exec -it <pod-name-ns1> -n ns1 -- cat /etc/resolv.conf
```
Example output:
```text
nameserver 10.96.0.10
search ns1.svc.cluster.local svc.cluster.local cluster.local
```
Construct the FQDN for the service in ns2:
```text
svc-ns2.ns2.svc.cluster.local
```
Test it:
```bash
kubectl exec -it <pod-name-ns1> -n ns1 -- sh
# curl svc-ns2.ns2.svc.cluster.local
```
✅ Result: Should now work across namespaces.
### 11. Clean up

Delete both namespaces and all resources inside them:
```bash
kubectl delete namespace ns1 ns2
```
📝 Key Takeaways
Concept	Explanation:
`Namespace` isolation	Resources in different namespaces are logically separated.
Pod-to-pod networking	Pod IPs are reachable across namespaces (unless NetworkPolicy restricts).
Service DNS	Short names (svc-name) only resolve within the same namespace.
FQDN	Use `svc-name.namespace.svc.cluster.local` for cross-namespace access.
Cleanup	Deleting a namespace deletes all resources inside it.

