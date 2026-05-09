# Kubernetes Secrets Hands on

This guide explains how to use a Kubernetes Secret (`tkb-secret`) and mount it into a Pod (`secret-pod`) using a volume.

---

## File Structure

```text
secrets/
├── secret.yaml
├── pod.yaml
└── README.md
```

---

# 1. secret.yaml

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tkb-secret

type: Opaque

data:
  username: bmlnZWxwb3VsdG9u
  password: UGFzc3dvcmQxMjM=
```

## Explanation

* `kind: Secret` → Defines a Kubernetes Secret object.
* `name: tkb-secret` → Name of the Secret.
* `type: Opaque` → Generic secret type (custom key/value data).
* `data:` → Contains base64 encoded values.

### Encoded values

| Key      | Base64 Value     | Plain Text   |
| -------- | ---------------- | ------------ |
| username | bmlnZWxwb3VsdG9u | nigelpoulton |
| password | UGFzc3dvcmQxMjM= | Password123  |

---

# 2. pod.yaml

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: secret-pod

spec:
  volumes:
  - name: secret-vol
    secret:
      secretName: tkb-secret

  containers:
  - name: secret-ctr
    image: nginx

    volumeMounts:
    - name: secret-vol
      mountPath: "/etc/tkb"
```

## Explanation

### Volume section

```yaml
volumes:
  - name: secret-vol
    secret:
      secretName: tkb-secret
```

* Creates a volume from the Secret `tkb-secret`.

---

### Container section

```yaml
volumeMounts:
  - name: secret-vol
    mountPath: "/etc/tkb"
```

* Mounts the Secret inside the container at `/etc/tkb`.

---

# 3. Apply Secret

```bash
kubectl apply -f secret.yaml
```

Expected output:

```text
secret/tkb-secret created
```

---

# 4. Apply Pod

```bash
kubectl apply -f pod.yaml
```

Expected output:

```text
pod/secret-pod created
```

---

# 5. Verify Pod

```bash
kubectl get pods
```

Wait until:

```text
STATUS: Running
```

---

# 6. Check Mounted Secret Files

```bash
kubectl exec secret-pod -- ls /etc/tkb
```

Output:

```text
password
username
```

Each key becomes a file.

---

# 7. Read Secret Values

### Username

```bash
kubectl exec secret-pod -- cat /etc/tkb/username
```

Output:

```text
nigelpoulton
```

### Password

```bash
kubectl exec secret-pod -- cat /etc/tkb/password
```

Output:

```text
Password123
```

---

# How It Works

1. Secret is created and stored in etcd.
2. Pod references the Secret.
3. kubelet fetches Secret.
4. Secret is mounted as files inside container.
5. Application reads values as normal text.

---

# File Mapping

| Secret Key | File Path         |
| ---------- | ----------------- |
| username   | /etc/tkb/username |
| password   | /etc/tkb/password |

---

# Important Security Note

* Kubernetes Secrets are **NOT encrypted by default**
* They are only **base64 encoded**

Decode example:

```bash
echo UGFzc3dvcmQxMjM= | base64 -d
```

Output:

```text
Password123
```

---

# Cleanup

```bash
kubectl delete -f pod.yaml
kubectl delete -f secret.yaml
```

---

# Summary

* Secret stores sensitive data in base64 form
* Pod mounts Secret as files under `/etc/tkb`
* Each key becomes a file
* Data is readable inside container as plain text
---
<img width="1148" height="598" alt="image" src="https://github.com/user-attachments/assets/e311b491-2588-411d-9ae3-aeb5c568178e" />
