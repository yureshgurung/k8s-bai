
# Kubernetes Multi-App Setup (Shield & Hydra)

##  Overview

This project deploys two applications in Kubernetes:

*  **shield app**
*  **hydra app**

Each app is exposed using:

* Service (ClusterIP / NodePort)
* Ingress (host + path-based routing)

---

#  Architecture

```
Browser
   ↓
Ingress Controller (10.10.10.202)
   ↓
Kubernetes Service (8080)
   ↓
Pod (8000)
   ↓
React App
```

---

#  Resources Created

## 1. Pods

* `shield` → runs app on port `8000`
* `hydra` → runs app on port `8000`

---

## 2. Services

* `svc-shield` → exposes shield pod
* `svc-hydra` → exposes hydra pod

| Service    | Port | Target Port |
| ---------- | ---- | ----------- |
| svc-shield | 8080 | 8000        |
| svc-hydra  | 8080 | 8000        |

---

## 3. Ingress

Ingress routes traffic based on:

###  Host-based routing

* `shield.mcu.com` → shield app
* `hydra.mcu.com` → hydra app

###  Path-based routing

* `mcu.com/shield` → shield app
* `mcu.com/hydra` → hydra app

---

# Deployment Steps

## 1. Apply Kubernetes resources

```bash
kubectl apply -f app.yaml
kubectl apply -f ingress.yml
```

---

## 2. Check pods

```bash
kubectl get pods
```

---

## 3. Check services

```bash
kubectl get svc
```

---

## 4. Check ingress

```bash
kubectl get ingress
```

---

#  Access Methods

## 🔹 Method 1: Ingress (Recommended)

First find ingress IP:

```bash
kubectl get svc -n ingress-nginx
```

Example IP:

```
10.10.10.202
```

Add to `/etc/hosts`:

```
10.10.10.202 shield.mcu.com
10.10.10.202 hydra.mcu.com
10.10.10.202 mcu.com
```


```
http://shield.mcu.com
http://hydra.mcu.com
http://mcu.com/shield
http://mcu.com/hydra
```

<img width="1487" height="510" alt="image" src="https://github.com/user-attachments/assets/ec2e8713-6a60-434e-8994-9f7beef401b2" />

---

##  Method 2: Port Forward (Testing only)

```bash
kubectl port-forward pod/shield 8000:8000 --address=0.0.0.0
```

Open:

```
curl localhost:8000
ipaddress:8000 , for example: 10.10.10.181:8000
```
<img width="1902" height="961" alt="image" src="https://github.com/user-attachments/assets/92bf6b94-6a01-4c40-8b91-99baa4a5be08" />

---

##  Method 3: NodePort

Check service:

```bash
kubectl get svc
```


Example:

```
svc-shield 8080:32449
```

Open:

```
http://<NODE-IP>:32449
```


<img width="1919" height="847" alt="image" src="https://github.com/user-attachments/assets/b2ce78ad-a052-493c-a3b0-173a71af69d3" />

---

#  Key Concepts

* **Pod** → runs application
* **Service** → connects traffic to pod
* **Ingress** → routes traffic using domain/path
* **NodePort** → exposes service externally
* **port-forward** → temporary local access


#  Test Commands

```bash
curl http://hydra.mcu.com
curl http://shield.mcu.com
```

---


<img width="1487" height="510" alt="image" src="https://github.com/user-attachments/assets/500b3052-9e6c-4c9f-95fd-f459ab0b78e6" />


