# Helm for Kubernetes — Complete Beginner Guide

![Helm Logo](https://helm.sh/img/helm.svg)

---

# 1. What Is Helm?

**Helm** is the package manager for Kubernetes.

It works like:

* `apt` for Ubuntu
* `yum` for RHEL
* `npm` for Node.js
* `pip` for Python

Helm helps you:

* Install applications into Kubernetes
* Upgrade applications
* Roll back to previous versions
* Manage configuration using values
* Reuse templates

---

# 2. Why Helm Is Needed

Without Helm, installing an application often requires many YAML files:

* Deployment
* Service
* ConfigMap
* Secret
* Ingress
* HPA
* ServiceAccount

Helm packages all of these into one reusable chart.

---

# 3. Real-World Analogy

Imagine installing WordPress manually:

* Create Deployment
* Create Service
* Create PVC
* Create Secrets
* Configure Ingress

With Helm:

```bash
helm install my-wordpress bitnami/wordpress
```

One command does everything.

---

# 4. Core Helm Concepts

## Chart

A Helm chart is a package containing Kubernetes templates.

## Release

A release is one installed instance of a chart.

## Repository

A location where charts are stored.

## Values

Configuration variables used to customize charts.

## Templates

YAML files with placeholders.

---

# 5. Chart vs Release

| Concept | Meaning            | Example         |
| ------- | ------------------ | --------------- |
| Chart   | Blueprint/template | `bitnami/nginx` |
| Release | Installed copy     | `my-nginx`      |

---

# 6. Helm Architecture

```text
Helm CLI
   |
   v
Chart Repository
   |
   v
Chart Templates + Values
   |
   v
Rendered Kubernetes YAML
   |
   v
Kubernetes API Server
```

---

# 7. Install Helm

## Linux

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

## Verify

```bash
helm version
```

---

# 8. Add Repositories

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

List repos:

```bash
helm repo list
```

---

# 9. Search Charts

```bash
helm search repo nginx
helm search repo wordpress
```

---

# 10. Install a Chart

```bash
helm install my-nginx bitnami/nginx
```

### Syntax

```bash
helm install <release-name> <chart>
```

---

# 11. Install in Specific Namespace

```bash
helm install my-nginx bitnami/nginx -n web --create-namespace
```

---

# 12. Check Releases

```bash
helm ls
helm ls -A
```

---

# 13. Check Release Status

```bash
helm status my-nginx
```

---

# 14. View Generated YAML

```bash
helm get manifest my-nginx
```

---

# 15. View User Values

```bash
helm get values my-nginx
```

---

# 16. Uninstall a Release

```bash
helm uninstall my-nginx
```

---

# 17. Upgrade a Release

```bash
helm upgrade my-nginx bitnami/nginx
```

---

# 18. Roll Back to Previous Revision

```bash
helm rollback my-nginx 1
```

---

# 19. Release History

```bash
helm history my-nginx
```

---

# 20. Create Your Own Chart

```bash
helm create mychart
```

This creates:

```text
mychart/
├── Chart.yaml
├── values.yaml
├── charts/
├── templates/
└── .helmignore
```

---

# 21. Chart.yaml

Chart metadata.

```yaml
apiVersion: v2
name: mychart
version: 0.1.0
appVersion: "1.0"
```

---

# 22. values.yaml

Default configuration values.

```yaml
replicaCount: 2
image:
  repository: nginx
  tag: alpine
service:
  type: ClusterIP
  port: 80
```

---

# 23. Templates

Example placeholder:

```yaml
replicas: {{ .Values.replicaCount }}
```

---

# 24. Template Variables

| Variable             | Meaning                 |
| -------------------- | ----------------------- |
| `.Values`            | Values from values.yaml |
| `.Release.Name`      | Release name            |
| `.Chart.Name`        | Chart name              |
| `.Chart.Version`     | Chart version           |
| `.Release.Namespace` | Namespace               |

---

# 25. Example Template

```yaml
metadata:
  name: {{ .Release.Name }}
```

If release name is `dev`, output becomes:

```yaml
metadata:
  name: dev
```

---

# 26. Override Values

```bash
helm install my-nginx bitnami/nginx --set replicaCount=3
```

---

# 27. Use Custom Values File

```bash
helm install my-nginx bitnami/nginx -f custom-values.yaml
```

---

# 28. Dry Run

```bash
helm install my-nginx bitnami/nginx --dry-run
```

---

# 29. Debug Mode

```bash
helm install my-nginx bitnami/nginx --dry-run --debug
```

---

# 30. Render YAML Without Installing

```bash
helm template my-nginx bitnami/nginx
```

---

# 31. Lint a Chart

```bash
helm lint mychart
```

---

# 32. Package a Chart

```bash
helm package mychart
```

Produces:

```text
mychart-0.1.0.tgz
```

---

# 33. Install Local Chart

```bash
helm install test ./mychart
```

---

# 34. Dependencies

Add to `Chart.yaml`:

```yaml
dependencies:
  - name: redis
    version: 18.0.0
    repository: https://charts.bitnami.com/bitnami
```

Download:

```bash
helm dependency update
```

---

# 35. Hooks

Run actions before/after install or upgrade.

Examples:

* pre-install
* post-install
* pre-upgrade
* post-upgrade

---

# 36. Common Helm Commands

| Command            | Purpose               |
| ------------------ | --------------------- |
| `helm repo add`    | Add repo              |
| `helm repo update` | Refresh repos         |
| `helm search repo` | Search charts         |
| `helm install`     | Install chart         |
| `helm ls -A`       | List releases         |
| `helm status`      | Show release status   |
| `helm upgrade`     | Upgrade release       |
| `helm rollback`    | Roll back             |
| `helm uninstall`   | Delete release        |
| `helm create`      | Create chart skeleton |
| `helm lint`        | Validate chart        |
| `helm template`    | Render YAML           |

---

# 37. Practical Example

## Install NGINX

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm install web bitnami/nginx
```

## Verify

```bash
helm ls
kubectl get all
```

## Upgrade

```bash
helm upgrade web bitnami/nginx --set replicaCount=3
```

## Rollback

```bash
helm rollback web 1
```

## Uninstall

```bash
helm uninstall web
```

---

# 38. Release Revisions

Every upgrade creates a new revision.

| Revision | Description     |
| -------- | --------------- |
| 1        | Initial install |
| 2        | Upgrade         |
| 3        | Another upgrade |

---

# 39. Namespace and Release Independence

The same chart can be installed multiple times:

```bash
helm install dev bitnami/nginx -n team-a
helm install prod bitnami/nginx -n team-b
```

---

# 40. Helm and Kubernetes Relationship

Helm does not replace Kubernetes.

Helm generates YAML and sends it to Kubernetes.

---

# 41. How Values Flow

```text
values.yaml
   +
custom-values.yaml
   +
--set options
   ↓
Templates
   ↓
Rendered YAML
```

---

# 42. Best Practices

* Use meaningful release names.
* Store custom values in files.
* Use `helm template` before install.
* Use `helm lint` for validation.
* Keep charts versioned.

---

# 43. Troubleshooting

## Release not found

```bash
helm ls -A
```

## Check status

```bash
helm status <release>
```

## View manifests

```bash
helm get manifest <release>
```

## Check pod logs

```bash
kubectl logs <pod>
```

---

# 44. Interview Questions

### What is Helm?

A package manager for Kubernetes.

### What is a chart?

A package containing templates.

### What is a release?

An installed instance of a chart.

### Difference between chart and release?

Chart is the template; release is the deployment.

### What is values.yaml?

Default configuration values.

### What does `helm upgrade` do?

Updates an existing release.

### What does `helm rollback` do?

Restores a previous revision.

---

# 45. Frequently Used Commands

```bash
helm version
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo nginx
helm install web bitnami/nginx
helm ls -A
helm status web
helm get values web
helm get manifest web
helm upgrade web bitnami/nginx --set replicaCount=3
helm history web
helm rollback web 1
helm uninstall web
helm create mychart
helm lint mychart
helm template test ./mychart
helm package mychart
```

---

# 46. Learning Path

1. Understand Kubernetes YAML.
2. Install existing charts.
3. Override values.
4. Create your own chart.
5. Learn templating.
6. Use dependencies and hooks.
7. Build CI/CD pipelines.

---

# 47. Summary

Helm simplifies Kubernetes application management by packaging multiple YAML files into reusable charts.

Key concepts:

* Chart = template package
* Release = installed instance
* Repository = chart store
* Values = customization
* Templates = dynamic YAML

With Helm, complex deployments can be installed, upgraded, rolled back, and removed using simple commands.

---

# 48. One-Sentence Definition

> Helm is the package manager for Kubernetes that uses charts to install and manage applications as versioned releases.
