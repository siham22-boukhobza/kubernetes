# Kubernetes Liveness Probes - Exec, HTTP & TCP

This lab demonstrates three types of **Kubernetes Liveness Probes**:
1. **`exec`** – Runs a command inside the container
2. **`httpGet`** – Performs an HTTP GET request
3. **`tcpSocket`** – Attempts to open a TCP connection

## 🎯 What you'll learn

- How liveness probes detect unhealthy containers
- The difference between `exec`, `httpGet`, and `tcpSocket` probes
- How Kubernetes automatically restarts failing containers
- Understanding `CrashLoopBackOff` and container restart behavior

---

## 📋 Prerequisites

- Kubernetes cluster (Kind recommended)
- `kubectl` configured

---

## 🚀 Lab 1: Exec Probe

**Manifest:** [`liveness-exec.yml`](./liveness-exec.yml)

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```
---

## 🚀 Lab 2: HTTP Get Probe

**Manifest:** [`liveness-http.yml`](./liveness-http.yml)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/e2e-test-images/agnhost:2.40
    args:
    - liveness
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 3
      periodSeconds: 3
 ```     

---

## 🚀 Lab 3: TCP Socket Probe

**Manifest:** [`liveness-tcp.yml`](./liveness-tcp.yml)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: tcp-pod
  labels:
    app: tcp-pod
spec:
  containers:
  - name: goproxy
    image: registry.k8s.io/goproxy:0.1
    ports:
    - containerPort: 8080
    livenessProbe:
      tcpSocket:
        port: 3000
      initialDelaySeconds: 10
      periodSeconds: 5
```
---
### Step 1: Deploy the pod
```bash
kubectl apply -f liveness-tcp.yml
kubectl get pods -w
```
### Step 2: Observe restarts

The probe tries to connect to port 3000, but the container listens on port 8080.
```bash
kubectl describe pod tcp-pod
```
Expected events:
```text
Warning  Unhealthy  Liveness probe failed: dial tcp 10.244.1.3:3000: connect: connection refused
Normal   Killing    Container goproxy failed liveness probe, will be restarted
```
Result: The container restarts repeatedly because port 3000 is not open.
###  3: Check restart count
```bash
kubectl get pod tcp-pod
```
Expected output:
```text
NAME      READY   STATUS    RESTARTS      AGE
tcp-pod   1/1     Running   4 (5s ago)    2m
```
### 💡 How it works
|Parameter|Value|Purpose|
|---------|-----|-------|
|port|3000|Port to check for TCP connection|
|initialDelaySeconds|10|Wait 10 seconds before first check|
|periodSeconds|5|Check every 5 seconds|

The probe is considered successful if it can open a TCP connection to the specified port.

### 📊 Liveness Probe Comparison

| Probe Type | Command | Use Case | Example |
|---|---|---|---|
| `exec` | Runs a command inside the container | Applications without HTTP endpoint | Checking a file exists |
| `httpGet` | Checks HTTP status code | Web services and APIs | `/healthz` endpoint |
| `tcpSocket` | Attempts to open a TCP connection | Databases, message queues, custom TCP services | Checking a port is open |
---

## 🧹 Clean Up

```bash
kubectl delete pod liveness-exec hello tcp-pod
```


