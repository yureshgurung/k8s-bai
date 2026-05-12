# Kubernetes Recreate Deployment Strategy

This project demonstrates how to use the Kubernetes **Recreate Deployment Strategy** to deploy and update an application.

The example uses the Docker image `amitabhdevops/online_shop` and exposes it through a NodePort Service.

---

## What Is the Recreate Strategy?

In Kubernetes, the **Recreate** deployment strategy terminates all existing Pods before creating new Pods.

### How It Works

1. Kubernetes deletes all currently running Pods.
2. The application becomes temporarily unavailable.
3. Kubernetes creates new Pods with the updated image or configuration.
4. The Service starts routing traffic to the new Pods.

### Key Characteristic

* **Downtime occurs** during every update.

### Best Use Cases

* Development and testing environments.
* Applications where downtime is acceptable.
* Applications that cannot run two versions simultaneously.
* Major upgrades involving database schema changes.

---

## Project Structure

```text
deployment-strategy/
├── namespace.yaml
├── deployment.yaml
└── service.yaml
```

---

## Cluster Information

This project was tested on a two-node Kubernetes cluster.

| Node Name      | Role                | Internal IP  |
| -------------- | ------------------- | ------------ |
| yuresh-master1 | control-plane, etcd | 10.10.10.181 |
| yuresh-worker1 | worker              | 10.10.10.182 |

Kubernetes Version: `v1.35.4+rke2r1`

---
<img width="1902" height="154" alt="image" src="https://github.com/user-attachments/assets/2705bf4e-752d-448f-9918-1b4746f6a4cc" />


# Namespace Manifest

## `namespace.yaml`

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: recreate-y
```

### Purpose

Creates a dedicated namespace named `recreate-y` to isolate all resources.

---

# Deployment Manifest

## `deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: notebai
  namespace: recreate-y
  labels:
    app: online-shop
spec:
  replicas: 4
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: online-shop
  template:
    metadata:
      labels:
        app: online-shop
    spec:
      containers:
      - name: online-shop
        image: amitabhdevops/online_shop
        resources:
          requests:
            cpu: "200m"
            memory: "256Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
```

### Deployment Explanation

| Field                      | Description                               |
| -------------------------- | ----------------------------------------- |
| `replicas: 4`              | Creates four Pods                         |
| `strategy.type: Recreate`  | Deletes all Pods before creating new ones |
| `selector.matchLabels`     | Selects Pods managed by this Deployment   |
| `template.metadata.labels` | Labels applied to Pods                    |
| `containers[].image`       | Docker image to run                       |
| `resources.requests`       | Minimum guaranteed CPU and memory         |
| `resources.limits`         | Maximum allowed CPU and memory            |

---

# Service Manifest

## `service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: recreate-service
  namespace: recreate-y
spec:
  selector:
    app: online-shop
  type: NodePort
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
      nodePort: 30000
```

### Service Explanation

| Field              | Description                             |
| ------------------ | --------------------------------------- |
| `selector`         | Matches Pods labeled `app: online-shop` |
| `type: NodePort`   | Exposes the application externally      |
| `port: 3000`       | Service port                            |
| `targetPort: 3000` | Container port                          |
| `nodePort: 30000`  | Port opened on every node               |

---

# Apply the Resources

```bash
kubectl apply -f namespace.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

---

# Verify Resources

## Check Namespace

```bash
kubectl get ns
```

## Check Deployment

```bash
kubectl get deployment -n recreate-y
```
<img width="1393" height="102" alt="image" src="https://github.com/user-attachments/assets/6b06aee5-786e-4e9d-87f1-9bcc1a718ac1" />


## Check Pods

```bash
kubectl get pods -n recreate-y
```
<img width="1141" height="168" alt="image" src="https://github.com/user-attachments/assets/23a616e0-38f4-4b1f-b680-41591d22f1bd" />


## Watch Pods in Real Time

```bash
watch kubectl get pods -n recreate-y
```

> This command refreshes the Pod list every 2 seconds so you can observe old Pods terminating and new Pods being created during a Recreate rollout.

## Alternative Watch Mode

```bash
kubectl get pods -n recreate-y -w
```

## Check Service

```bash
kubectl get svc -n recreate-y
```

---

# Access the Application

Because the Service type is `NodePort`, the application is available on all cluster nodes.

## URLs

* `http://10.10.10.181:30000`
* `http://10.10.10.182:30000`

---
<img width="1919" height="672" alt="image" src="https://github.com/user-attachments/assets/b2a720b2-01e3-4875-876a-9df6512abe87" />


# Optional Port Forward

```bash
kubectl port-forward --address 0.0.0.0 svc/recreate-service 3000:3000 -n recreate-y
```

Then open:

* `http://10.10.10.181:3000`

> Port forwarding is temporary and works only while the command is running.

---

# Update the Application Image

```bash
kubectl set image deployment/notebai \
  online-shop=amitabhdevops/online_shop_without_footer \
  -n recreate-y
```
<img width="1021" height="179" alt="image" src="https://github.com/user-attachments/assets/9162a0df-af6d-4df0-a6d0-0cfc37ad8020" />

---

# Observe the Recreate Rollout

Use one of the following commands:

```bash
watch kubectl get pods -n recreate-y
```

or

```bash
kubectl get pods -n recreate-y -w
```

### What You Will See

1. All existing Pods are terminated.
2. There is a short period where no Pods are running.
3. New Pods are created.
4. New Pods become `Running`.

---

# Check Rollout Status

```bash
kubectl rollout status deployment/notebai -n recreate-y
```

Expected output:

```text
deployment "notebai" successfully rolled out
```

---

# Verify Current Image

```bash
kubectl get deployment notebai -n recreate-y \
  -o jsonpath='{.spec.template.spec.containers[0].image}'
echo
```
<img width="1822" height="72" alt="image" src="https://github.com/user-attachments/assets/3c753a97-94c1-4510-a6f4-b17d0847662c" />

---

# Rollback to Previous Version

```bash
kubectl rollout undo deployment/notebai -n recreate-y
```
<img width="1827" height="97" alt="image" src="https://github.com/user-attachments/assets/067b94c9-8ff6-4130-a750-6fd273943bfe" />

---

# View Rollout History

```bash
kubectl rollout history deployment/notebai -n recreate-y
```

---

# Recreate Strategy Flow

```text
Old Pods Running (4)
        ↓
All Pods Deleted
        ↓
Downtime Occurs
        ↓
New Pods Created
        ↓
Application Available Again
```

---

# Advantages

* Very simple deployment strategy.
* Ensures only one application version runs at a time.
* Useful for incompatible versions.

---

# Disadvantages

* Causes downtime.
* Not suitable for zero-downtime production systems.

---

# Recreate vs RollingUpdate

| Feature                           | Recreate | RollingUpdate |
| --------------------------------- | -------- | ------------- |
| Downtime                          | Yes      | Usually No    |
| Old and New Versions Run Together | No       | Yes           |
| Complexity                        | Simple   | Moderate      |
| Production Suitability            | Limited  | Excellent     |

---

# Useful Commands Summary

```bash
# Apply resources
kubectl apply -f namespace.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# View resources
kubectl get all -n recreate-y

# Watch Pods
watch kubectl get pods -n recreate-y

# Update image
kubectl set image deployment/notebai \
  online-shop=amitabhdevops/online_shop_without_footer \
  -n recreate-y

# Check rollout status
kubectl rollout status deployment/notebai -n recreate-y

# Rollback
kubectl rollout undo deployment/notebai -n recreate-y

# Delete namespace
kubectl delete namespace recreate-y
```

---

# Cleanup

```bash
kubectl delete namespace recreate-y
```

---

# Key Learning Points

* `strategy.type: Recreate` deletes all existing Pods before creating new ones.
* `NodePort` exposes the application on every cluster node.
* `watch kubectl get pods -n recreate-y` is the easiest way to observe rollout behavior.
* `kubectl set image` updates the container image.
* `kubectl rollout undo` restores the previous version.

---

# Author

GitHub: `yuresh gurung`

