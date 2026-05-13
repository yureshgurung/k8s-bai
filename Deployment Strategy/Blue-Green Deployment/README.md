# Kubernetes Blue-Green Deployment Strategy (online-shop)

This project demonstrates the **Blue-Green Deployment Strategy** in Kubernetes using an `online-shop` application.

Blue-Green deployment is a release strategy that maintains two identical environments:

* **Blue Environment** → Current production version
* **Green Environment** → New version to be tested and validated

Once the Green version is verified, traffic is switched instantly from Blue to Green.

---

## Overview

Blue-Green deployment minimizes deployment risk and enables **instant rollback**.

### How It Works

1. **Blue Environment** is serving live user traffic.
2. **Green Environment** is deployed with the new application version.
3. Green is tested independently.
4. The Service selector is updated to route traffic to Green.
5. If issues occur, switch the selector back to Blue.

---
<img width="1618" height="972" alt="bg" src="https://github.com/user-attachments/assets/db51ac3f-aa14-4c5e-9847-50130c79611d" />


##  Pros and Cons

| Pros                         | Cons                                     |
| ---------------------------- | ---------------------------------------- |
| Instant rollout and rollback | Requires double infrastructure resources |
| No downtime                  | Higher operational cost                  |
| Easy testing before release  | Full platform testing is required        |
| Low deployment risk          | Slightly more complex setup              |

---

> **Recommended for:** Production environments where reliability and instant rollback are important.

---

##  Cluster Information

This project was tested on a two-node Kubernetes cluster.

| Node Name      | Role                | Internal IP  |
| -------------- | ------------------- | ------------ |
| yuresh-master1 | control-plane, etcd | 10.10.10.181 |
| yuresh-worker1 | worker              | 10.10.10.182 |

**Kubernetes Version:** `v1.35.4+rke2r1`

---

##  Project Files

```text
bg-namespace.yaml
blue.yaml
green.yaml
```

---

## Namespace

### `bg-namespace.yaml`

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: blue-green-y
```

### Apply Namespace

```bash
kubectl apply -f bg-namespace.yaml
```

---

## Blue Environment

### `blue.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: online-shop-blue
  namespace: blue-green-y
  labels:
    app: online-shop-blue
spec:
  replicas: 4
  selector:
    matchLabels:
      app: online-shop-blue
  template:
    metadata:
      labels:
        app: online-shop-blue
    spec:
      containers:
      - name: online-shop-blue
        image: amitabhdevops/online_shop_without_footer
        resources:
          requests:
            cpu: "200m"
            memory: "256Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
---
apiVersion: v1
kind: Service
metadata:
  name: online-shop-blue-deployment-service
  namespace: blue-green-y
spec:
  selector:
    app: online-shop-blue
  type: NodePort
  ports:
    - port: 3002
      targetPort: 3000
      nodePort: 30002
```

---

##  Green Environment

### `green.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: online-shop-green
  namespace: blue-green-y
  labels:
    app: online-shop-green
spec:
  replicas: 4
  selector:
    matchLabels:
      app: online-shop-green
  template:
    metadata:
      labels:
        app: online-shop-green
    spec:
      containers:
      - name: online-shop-green
        image: amitabhdevops/online_shop
        resources:
          requests:
            cpu: "200m"
            memory: "256Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
---
apiVersion: v1
kind: Service
metadata:
  name: online-shop-green-deployment-service
  namespace: blue-green-y
spec:
  selector:
    app: online-shop-green
  type: NodePort
  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 30000
```

---

##  Deploy Both Environments

```bash
kubectl apply -f blue.yaml
kubectl apply -f green.yaml
```

---

## 👀 Monitor Pods

```bash
kubectl get pods -n blue-green-y -w
```

---

##  Access Applications

### Blue Environment

```text
http://10.10.10.181:30002
http://10.10.10.182:30002
```
<img width="1913" height="873" alt="image" src="https://github.com/user-attachments/assets/82c6d970-3974-44a6-aa1e-b77a45192001" />

### Green Environment

```text
http://10.10.10.181:30000
http://10.10.10.182:30000
```
<img width="1918" height="910" alt="image" src="https://github.com/user-attachments/assets/d9757c8b-b21a-43e7-954a-64daffc01d57" />

---

##  Optional Port Forward

### Blue Service

```bash
kubectl port-forward --address 0.0.0.0 svc/online-shop-blue-deployment-service 30002:3002 -n blue-green-y
```

Access:

```text
http://10.10.10.181:30002
```

### Green Service

```bash
kubectl port-forward --address 0.0.0.0 svc/online-shop-green-deployment-service 30000:3000 -n blue-green-y
```

Access:

```text
http://10.10.10.181:30000
```

---

## Switch Traffic from Blue to Green

Edit `blue.yaml` and change the Service selector.

### Before

```yaml
selector:
  app: online-shop-blue
```

### After

```yaml
selector:
  app: online-shop-green
```

Apply the change:

```bash
kubectl apply -f blue.yaml
```

Now all traffic hitting:

```text
http://10.10.10.181:30002
```
<img width="1912" height="963" alt="image" src="https://github.com/user-attachments/assets/20198cb4-cffe-4750-9e0d-eba51c127f86" />


will be served by the **Green Deployment**.

---

##  Instant Rollback

If the Green version has problems, revert the selector back to:

```yaml
selector:
  app: online-shop-blue
```

Apply again:

```bash
kubectl apply -f blue.yaml
```

Rollback is immediate.

---

## Stop All Port Forward Sessions

```bash
pkill -f "kubectl port-forward"
```

---

##  Useful Commands

### Check Deployments

```bash
kubectl get deployments -n blue-green-y
```

### Check Services

```bash
kubectl get svc -n blue-green-y
```

### Rollout Status

```bash
kubectl rollout status deployment online-shop-green -n blue-green-y
```

---

## Key Learning

* Blue and Green environments run simultaneously.
* Traffic switching is controlled by the Service selector.
* Rollback is instant.
* No downtime occurs during release.

---

##  Final Summary

This project demonstrates:

* Kubernetes Namespace isolation
* Blue-Green deployment strategy
* NodePort-based access
* Traffic switching via Service selector
* Instant rollback capability
* Real-time monitoring with `kubectl get pods -w`

---

                                                                ⭐ 
