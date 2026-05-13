# Deployment Strategies 

The major deployment strategies used in modern DevOps and Kubernetes environments.

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


## 1. Zero-Downtime Deployment Strategies (Production-Ready)

### 🔵 Rolling Update

Gradually replaces old application versions with new ones without downtime.

**How it works:**

* Replaces pods incrementally
* Old and new versions run simultaneously during transition

**Pros:**

* No downtime
* Easy to implement in Kubernetes

**Cons:**

* Slow rollback
* Version mismatch during rollout

**Best for:** Standard production deployments

---

### 🔵 Blue-Green Deployment

Two identical environments: Blue (live) and Green (new version).

**How it works:**

* Deploy new version to Green
* Switch traffic from Blue → Green instantly

**Pros:**

* Instant rollback
* Zero downtime switch

**Cons:**

* High infrastructure cost
* Requires duplicate environments

**Best for:** Mission-critical applications

---

### 🔵 Canary Deployment

New version released to a small subset of users first.

**How it works:**

* Deploy to small % of traffic
* Gradually increase rollout if stable

**Pros:**

* Reduced risk
* Real user validation

**Cons:**

* Complex traffic management

**Best for:** Risk-sensitive production systems

---

### 🔵 Progressive Delivery

Advanced version of canary with automated analysis.

**How it works:**

* Gradual rollout with metrics monitoring
* Auto rollback on failure signals

**Pros:**

* Highly automated
* Safer than manual canary

**Cons:**

* Requires observability tools

**Best for:** Enterprise CI/CD pipelines

---

## 2. Downtime-Based Deployment

### 🔴 Recreate Deployment

Stops old version before starting new one.

**How it works:**

* Terminate all existing pods
* Deploy new version from scratch

**Pros:**

* Simple
* Easy to configure

**Cons:**

* Application downtime

**Best for:** Development / testing environments only

---

## 3. Validation-Based Deployment Strategies

### 🟢 Shadow Deployment

New version runs alongside production but does not serve users.

**How it works:**

* Duplicate traffic is sent to new version
* No user impact

**Pros:**

* Real-world testing
* Zero user risk

**Cons:**

* High resource usage

**Best for:** Load testing and validation

---

### 🟢 A/B Testing

Two versions served to different user groups.

**How it works:**

* Users split between versions A and B
* Metrics compared for decision-making

**Pros:**

* Data-driven UX decisions

**Cons:**

* Requires analytics setup

**Best for:** UI/UX experimentation

---

### 🟢 Feature Flags

Features are enabled/disabled without redeploying.

**How it works:**

* Code deployed with flags
* Features toggled dynamically

**Pros:**

* Instant control
* Decouples deploy from release

**Cons:**

* Complexity in code management

**Best for:** Continuous delivery systems

---

## 4. Quick Decision Guide

* Need zero downtime → Rolling Update / Blue-Green / Canary / Progressive Delivery
* Need fastest rollback → Blue-Green or Progressive Delivery
* Need safe testing in production → Canary or Shadow
* Need UX experimentation → A/B Testing
* Need continuous feature control → Feature Flags
* Simple dev setup → Recreate

---


