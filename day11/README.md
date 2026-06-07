# Kubernetes HPA - Command Reference

## 📋 Prerequisites

- Metrics Server installed (see day10 lab)

---

## 🚀 Commands executed

### 1. Deploy the application

```bash
kubectl apply -f deploy.yml
```
### 2. Verify deployment
bash
kubectl get pods
kubectl get deployment
kubectl get service
```
### 3. Create HPA (Horizontal Pod Autoscaler)
```bash
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
```
### 4. Check HPA status
```bash
kubectl get hpa
kubectl get hpa -w
```
### 5. Generate load
```bash
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
```
### 6. Watch scaling in action
```bash
kubectl get pods -w
kubectl get hpa -w
```
### 7. Stop load generation

Press Ctrl+C in the load-generator terminal
### 8. Clean up
```bash
kubectl delete hpa php-apache
kubectl delete deployment php-apache
kubectl delete service php-apache
```
### 📊 Verification commands
```bash
# Check node CPU metrics
kubectl top nodes

# Check pod CPU metrics
kubectl top pods

# Describe HPA details
kubectl describe hpa php-apache

# Watch HPA in real-time
kubectl get hpa php-apache --watch
```
### 🏆 Expected output
Scale up observed:
```text
NAME         REFERENCE               TARGETS     MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   cpu: 233%   1         10        4          62s
php-apache   Deployment/php-apache   cpu: 58%    1         10        5          75s
```
Scale down observed (5-10 minutes after stopping load):
```text
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   cpu: 0%    1         10        6          10m
php-apache   Deployment/php-apache   cpu: 0%    1         10        4          11m
php-apache   Deployment/php-apache   cpu: 0%    1         10        1          13m
```
### 🔑 Key takeaway

HPA automatically scales pods based on CPU utilization:

    Scale up when CPU exceeds 50% (takes ~1 minute)

    Scale down when CPU drops below 50% (takes ~5 minutes to prevent flapping)









