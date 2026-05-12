
# Kubernetes Rolling-update Deployment

This project demonstrates a **Rolling Update Deployment Strategy** in Kubernetes using a sample `online-shop` application.

---

## Overview

A **Rolling Update** is a Kubernetes deployment strategy where existing Pods are replaced with new Pods **gradually (one by one or in batches)**.

This ensures the application remains available with **zero downtime** during updates.

---
<img width="1536" height="1024" alt="rollingupdate" src="https://github.com/user-attachments/assets/d1f6d060-e55d-42e9-895f-503dc0913817" />


##  How Rolling Update Works

1. Kubernetes creates a new Pod with the updated image
2. The new Pod must pass readiness checks
3. Old Pods are terminated one by one
4. Process continues until all Pods are updated

---

##  Pros and Cons

| Pros                     | Cons                                       |
| ------------------------ | ------------------------------------------ |
| Zero downtime deployment | Rollout takes time                         |
| Safe production updates  | Rollback is slower than instant strategies |
| Gradual version upgrade  | No direct traffic splitting control        |
| High availability        | Temporary extra resource usage             |

---

##  Project Structure

```
namespace.yaml
rolingdeploy.yaml
rolingservice.yaml
```

---

##  Namespace
### Apply

```bash
kubectl apply -f namespace.yaml
```

---

## Deployment

```

### Apply Deployment

```bash
kubectl apply -f rolingdeploy.yaml
```

---

## Service (NodePort)

### Apply Service

```bash
kubectl apply -f rolingservice.yaml
```

---


Access the Application
Because the Service type is NodePort, the application is available on all cluster nodes.

URLs
http://10.10.10.181:30000

http://10.10.10.182:30000
image
Optional Port Forward

kubectl port-forward --address 0.0.0.0 svc/recreate-service 3000:3000 -n recreate-y
Then open:
<img width="1911" height="928" alt="image" src="https://github.com/user-attachments/assets/bdc8828c-f3cf-4eba-90fd-0c372098dd0a" />

http://10.10.10.181:3000

Port forwarding is temporary and works only while the command is running.

## Check Resources

### Pods

```bash
kubectl get pods -n rolling-y
```

### Live Watch Pods

```bash
kubectl get pods -n rolling-y -w
```

### Service

```bash
kubectl get svc -n rolling-y
```

---
##  What happens if image is wrong?

If you provide an invalid image like:

```yaml
image: wrong_image_name
```
<img width="1172" height="208" alt="image" src="https://github.com/user-attachments/assets/a7ba6c21-c501-49f7-a2bb-ca970bc6ec2f" />


###  outcomes:

### ImagePullBackOff

* Kubernetes cannot find the image
* 
## Rolling Update in Action

When you change the image in the deployment:

```yaml
image: amitabhdevops/online_shop_without_footer
```

and apply again:
<img width="1159" height="911" alt="image" src="https://github.com/user-attachments/assets/08db6dad-e4b2-4e8b-b6b4-4269375f4cb7" />


```bash
kubectl apply -f rolingdeploy.yaml
```

 Kubernetes will:

* Create new Pods with new image
* Terminate old Pods gradually
* Keep application running (zero downtime)

Check rollout:

```bash
kubectl rollout status deployment online-shop -n rolling-y
```

---



Check with:

```bash
kubectl get pods -n rolling-y
kubectl describe pod <pod-name> -n rolling-y
```

---

---

###  Result

* New Pods will NOT become Ready
* Rolling Update will pause
* Old Pods will continue running

---

##  Key Learning

* Rolling Update ensures safe deployments
* Kubernetes replaces Pods gradually
* Bad images can block rollout but do not break existing service

---

##  Final Summary

This project demonstrates:

✔ Kubernetes Namespace usage
✔ Rolling Update deployment strategy
✔ NodePort service exposure
✔ Real-time pod updates
✔ Handling deployment failures (bad image scenario)

---

