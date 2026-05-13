# Canary Deployment Strategy in Kubernetes

## Overview

A **Canary Deployment** in Kubernetes is a release strategy where a new version of an application is gradually introduced to a small subset of users before rolling it out to everyone. This approach helps reduce risk by exposing the new version to limited traffic first and monitoring its behavior.

---
<img width="1564" height="1006" alt="cannary" src="https://github.com/user-attachments/assets/e89b976c-4538-42fd-aeca-1bdf079a6f44" />

## How It Works

### 1. Deploy New Version (Canary Release)

The new version of the application is deployed alongside the stable version but receives only a small portion of traffic. This subset is called the **canary**.

* Old version continues serving most users
* New version gets limited exposure (e.g., 5–10%)

---

### 2. Traffic Splitting

Traffic is distributed between:

* **Stable version (v1)**
* **Canary version (v2)**

Initially:

* 90–95% → Stable
* 5–10% → Canary

Traffic distribution is often achieved using replica ratios or service mesh/ingress rules.

---

### 3. Monitoring and Analysis

The canary version is closely monitored for:

* Error rates
* Latency/performance
* Crash reports
* Resource usage

If issues are detected, rollback is immediate.

---

### 4. Gradual Rollout

If the canary performs well:

* Increase traffic step-by-step (10% → 25% → 50% → 100%)
* Eventually replace the stable version completely

This reduces risk during production releases.

---

## Advantages and Disadvantages

| Advantages                              | Disadvantages                 |
| --------------------------------------- | ----------------------------- |
| Reduced risk for production deployments | Slower rollout process        |
| Real user testing in production         | Requires monitoring setup     |
| Easy rollback to stable version         | Traffic tuning can be complex |
| Early detection of bugs                 | Infrastructure overhead       |

---

## When to Use

* Production environments
* Critical applications
* Systems requiring high reliability

---


## Setup Overview

### 1. Install Ingress Controller (Kind)

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/kind/deploy.yaml
```

Wait for readiness:

```bash
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s
```

If stuck in Pending state:

```bash
kubectl patch deployment ingress-nginx-controller -n ingress-nginx --type=json \
  -p='[{"op": "remove", "path": "/spec/template/spec/nodeSelector"}]'
```

---

### 2. Create Namespace

```bash
kubectl apply -f canary-namespace.yml
```

---

### 3. Deploy Applications

* **v1 (Stable version)** → 4 replicas
* **v2 (Canary version)** → 1 replica

```bash
kubectl apply -f canary-v1-deployment.yaml
kubectl apply -f canary-v2-deployment.yaml
```

---

### 4. Create Service

```bash
kubectl apply -f canary-combined-service.yaml
```

* Service selects pods using `app: online-shop`
* Does not differentiate version labels

---

### 5. Create Ingress

```bash
kubectl apply -f ingress.yaml
```

---

## Traffic Distribution Logic

Traffic is distributed based on replica count:

* v1 → 4 pods ≈ 80% traffic
* v2 → 1 pod ≈ 20% traffic

---

## Testing Canary Deployment

Port forward ingress controller:

```bash
kubectl port-forward -n ingress-nginx svc/ingress-nginx-controller 8080:80 --address 0.0.0.0 &
```

Access:

```
http://<INSTANCE_IP>:8080
```

Expected behavior:

* v1 appears ~80% of requests
* v2 appears ~20% of requests

---

## Adjusting Traffic Split

### Increase Canary Traffic

```bash
kubectl scale deployment online-shop-v1 -n canary-ns --replicas=3
kubectl scale deployment online-shop-v2 -n canary-ns --replicas=2
```

### More Canary Exposure

```bash
kubectl scale deployment online-shop-v1 -n canary-ns --replicas=2
kubectl scale deployment online-shop-v2 -n canary-ns --replicas=3
```

### Full Rollout to v2

```bash
kubectl scale deployment online-shop-v1 -n canary-ns --replicas=0
kubectl scale deployment online-shop-v2 -n canary-ns --replicas=5
```

---

## Monitoring

```bash
kubectl get pods -n canary-ns
kubectl get pods -n canary-ns --show-labels
kubectl describe svc online-shop-service -n canary-ns
watch kubectl get pods -n canary-ns
```

---

## Cleanup

Delete Kind cluster:

```bash
kind delete cluster --name dep-strg
```

---

## Key Takeaway

Canary Deployment is a **safe production release strategy** that minimizes risk by gradually exposing new versions to real users while monitoring system behavior.
