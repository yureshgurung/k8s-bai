# Kubernetes RBAC Practical 

This repository demonstrates a real-world example of Kubernetes RBAC (Role-Based Access Control).

## Scenario

You have a Kubernetes cluster with multiple namespaces:

* `dev` – Development environment
* `prod` – Production environment

You want to create a user named `ram` with the following permissions:

* Can create, update, and delete Deployments
* Can create and delete Pods
* Can only work in the `dev` namespace
* Cannot access the `prod` namespace
* Cannot view Secrets

---
## Project Structure

```text
.
├── developer-role.yaml
├── developer-binding.yaml
└── README.md
```

## RBAC Architecture

Kubernetes RBAC is based on three concepts:

1. **User** – The authenticated identity (`ram`)
2. **Role** – A set of allowed actions
3. **RoleBinding** – Assigns a Role to a User

### Flow

```text
User ram
   ↓
RoleBinding developer-binding
   ↓
Role developer-role
   ↓
Permissions granted in dev namespace
```

---

## Step 1: Create Namespaces

```bash
kubectl create namespace dev
kubectl create namespace prod
```

---

## Step 2: Create Role

Create a file named `developer-role.yaml`.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: developer-role
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]

- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch", "create", "delete"]
```

Apply the Role:

```bash
kubectl apply -f developer-role.yaml
```

---

## Step 3: Create RoleBinding

Create a file named `developer-binding.yaml`.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-binding
  namespace: dev
subjects:
- kind: User
  name: ram
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer-role
  apiGroup: rbac.authorization.k8s.io
```

Apply the RoleBinding:

```bash
kubectl apply -f developer-binding.yaml
```

---

## Permissions Granted to `ram`

### Allowed

```bash
kubectl get deployments -n dev
kubectl create deployment nginx --image=nginx -n dev
kubectl delete pod my-pod -n dev
```

### Denied

```bash
kubectl get secrets -n dev
kubectl get pods -n prod
kubectl get nodes
```

---

## Test Permissions

```bash
kubectl auth can-i create deployment --as=ram -n dev
kubectl auth can-i get secrets --as=ram -n dev
kubectl auth can-i get pods --as=ram -n prod
```

Expected output:

```text
yes
no
no
```

---

---

## Role vs RoleBinding

| Object      | Purpose                       |
| ----------- | ----------------------------- |
| Role        | Defines permissions           |
| RoleBinding | Assigns permissions to a user |

---

## Namespaced vs Cluster-Wide RBAC

| Resource           | Scope            |
| ------------------ | ---------------- |
| Role               | Single namespace |
| RoleBinding        | Single namespace |
| ClusterRole        | Entire cluster   |
| ClusterRoleBinding | Entire cluster   |

---

## Useful Commands

List RBAC resources:

```bash
kubectl get roles -A
kubectl get rolebindings -A
kubectl get clusterroles
kubectl get clusterrolebindings
```

Describe a Role:

```bash
kubectl describe role developer-role -n dev
```

Describe a RoleBinding:

```bash
kubectl describe rolebinding developer-binding -n dev
```

---

## Common Error

```text
Error from server (Forbidden): deployments.apps is forbidden:
User "ram" cannot create resource "deployments" in namespace "prod"
```

This means RBAC is working as intended.

---

<img width="1466" height="395" alt="image" src="https://github.com/user-attachments/assets/e0ad41b2-4cd7-4bc5-965a-98bf34d59481" />

