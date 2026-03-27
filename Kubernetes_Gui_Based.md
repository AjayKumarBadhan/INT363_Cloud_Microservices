# 🚀 Nginx Kubernetes Deployment

A production-ready Kubernetes configuration that deploys an Nginx web server using a `Deployment` resource and exposes it externally via a `NodePort` Service. This setup ensures high availability, resource governance, and automatic health monitoring out of the box.

---

## 📋 Table of Contents

- [Full Configuration Code](#-full-configuration-code)
- [Overview](#-overview)
- [Prerequisites](#-prerequisites)
- [Getting Started](#-getting-started)
- [Detailed Explanation](#-detailed-explanation)
  - [Deployment Resource](#1-deployment-resource)
  - [Metadata](#2-metadata)
  - [Replicas and Selector](#3-replicas-and-selector)
  - [Pod Template](#4-pod-template)
  - [Resource Requests and Limits](#5-resource-requests-and-limits)
  - [Liveness Probe](#6-liveness-probe)
  - [Readiness Probe](#7-readiness-probe)
  - [Service Resource](#8-service-resource)
  - [Traffic Flow](#9-traffic-flow)
- [Reference Tables](#-reference-tables)
- [Notes](#-notes)
- [License](#-license)

---

## 📄 Full Configuration Code

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "256Mi"
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 20
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30007
```

---

## 🧭 Overview

This YAML file defines **two Kubernetes resources** separated by `---`:

| Resource | Name | Purpose |
|---|---|---|
| `Deployment` | `nginx-deployment` | Manages 2 replicas of an Nginx pod |
| `Service` | `nginx-service` | Exposes Nginx externally on port `30007` |

Together they form a complete, self-healing web server deployment that automatically restarts unhealthy containers and load-balances traffic across all running pods.

---

## ✅ Prerequisites

- A running Kubernetes cluster (e.g., [Minikube](https://minikube.sigs.k8s.io/), [Kind](https://kind.sigs.k8s.io/), or a cloud provider like GKE/EKS/AKS)
- [`kubectl`](https://kubernetes.io/docs/tasks/tools/) installed and configured
- Access to Docker Hub (to pull `nginx:latest`)

---

## 🚀 Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
```

### 2. Apply the Configuration

```bash
kubectl apply -f nginx-deployment.yaml
```

### 3. Verify Everything is Running

```bash
# Check pod status
kubectl get pods

# Check deployment status
kubectl get deployment nginx-deployment

# Check service
kubectl get svc nginx-service
```

Expected output:

```
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-xxxx-yyyy          1/1     Running   0          30s
nginx-deployment-xxxx-zzzz          1/1     Running   0          30s
```

### 4. Access the Application

```bash
# On Minikube
minikube service nginx-service

# Or directly via NodePort (replace <NODE_IP> with your node's IP)
http://<NODE_IP>:30007
```

### 5. Tear Down

```bash
kubectl delete -f nginx-deployment.yaml
```

---

## 🔍 Detailed Explanation

### 1. Deployment Resource

```yaml
apiVersion: apps/v1
kind: Deployment
```

The file opens by declaring the API version and resource kind.

- **`apiVersion: apps/v1`** — Specifies the Kubernetes API group. `apps/v1` is the stable, production-recommended version for workloads like Deployments.
- **`kind: Deployment`** — Tells Kubernetes we want to create a `Deployment` object. A Deployment is the standard controller for running stateless applications — it continuously watches the cluster and ensures the desired number of pod replicas are always running.

---

### 2. Metadata

```yaml
metadata:
  name: nginx-deployment
  labels:
    app: nginx
```

Metadata identifies and organizes the resource within the cluster.

- **`name: nginx-deployment`** — The unique name of this Deployment within its namespace. Used by `kubectl` commands (e.g., `kubectl describe deployment nginx-deployment`).
- **`labels: app: nginx`** — A key-value pair attached to the resource. Labels are used for grouping, filtering, and selecting resources. The `app: nginx` label here is later referenced by the Service to route traffic to the correct pods.

---

### 3. Replicas and Selector

```yaml
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
```

This section defines *how many* pods to run and *which* pods this Deployment manages.

- **`replicas: 2`** — Instructs Kubernetes to maintain exactly 2 running instances (pods) of the Nginx container at all times. If a node crashes or a pod dies, Kubernetes automatically schedules a replacement on a healthy node.
- **`selector.matchLabels: app: nginx`** — The Deployment uses this label selector to identify which pods belong to it. Only pods with the label `app: nginx` are managed by this Deployment. This must match the labels defined in the Pod template below.

---

### 4. Pod Template

```yaml
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

The `template` is the blueprint that Kubernetes uses to create each pod.

- **`template.metadata.labels: app: nginx`** — Every pod created from this template receives the label `app: nginx`. This is what the `selector` above matches against, and also what the Service uses to route traffic.
- **`containers[].name: nginx`** — A human-readable name for the container within the pod. Useful for logs and debugging.
- **`image: nginx:latest`** — The Docker image to run. Kubernetes pulls this from Docker Hub. `latest` always fetches the most recent version (in production, prefer a pinned version like `nginx:1.25.3`).
- **`containerPort: 80`** — Declares that this container listens on port 80 (standard HTTP). This is informational for documentation and tooling — it does not create any network rules by itself.

---

### 5. Resource Requests and Limits

```yaml
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "256Mi"
```

Resource management is critical in a shared cluster. This section sets a guaranteed minimum and an enforced maximum for the container.

#### Requests
Requests are used by the **Kubernetes scheduler** to decide which node a pod can be placed on. The node must have at least this much free capacity.

| Resource | Value | Meaning |
|---|---|---|
| `cpu` | `100m` | 100 millicores = 0.1 CPU core guaranteed |
| `memory` | `128Mi` | 128 Mebibytes of RAM guaranteed |

#### Limits
Limits are enforced at runtime. The container cannot exceed these values.

| Resource | Value | Behaviour when exceeded |
|---|---|---|
| `cpu` | `500m` | Container is CPU-throttled (not killed) |
| `memory` | `256Mi` | Container is **OOMKilled** (restarted) |

> 💡 **Best Practice:** Always set both requests and limits. Without them, a single misbehaving pod can starve other workloads on the same node.

---

### 6. Liveness Probe

```yaml
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 20
```

The liveness probe answers the question: **"Is this container still alive and functioning?"**

Kubernetes sends an HTTP GET request to `http://localhost:80/` inside the container.

| Parameter | Value | Meaning |
|---|---|---|
| `initialDelaySeconds` | `15` | Wait 15 seconds after the container starts before the first check |
| `periodSeconds` | `20` | Repeat the check every 20 seconds |

**What happens on failure?** If the probe fails a configurable number of times (default: 3 consecutive failures), Kubernetes considers the container dead and **restarts it**. This is useful for catching scenarios like infinite loops, deadlocks, or corrupted application state that leave a process running but non-functional.

---

### 7. Readiness Probe

```yaml
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
```

The readiness probe answers the question: **"Is this container ready to receive traffic?"**

It uses the same HTTP GET mechanism but with different timing.

| Parameter | Value | Meaning |
|---|---|---|
| `initialDelaySeconds` | `5` | Begin checking 5 seconds after container starts |
| `periodSeconds` | `10` | Repeat check every 10 seconds |

**What happens on failure?** Unlike the liveness probe, a failed readiness probe does **not restart** the container. Instead, Kubernetes temporarily removes the pod from the Service's load balancer endpoint list. Traffic stops flowing to it until it passes again.

> 🔑 **Key Difference:**
> | Probe | On Failure | Use Case |
> |---|---|---|
> | **Liveness** | Restart the container | Detect dead/stuck processes |
> | **Readiness** | Stop sending traffic | Detect pods not yet ready to serve |

---

### 8. Service Resource

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30007
```

A **Service** is a stable networking abstraction that exposes a set of pods. Since pod IPs change every time a pod restarts, the Service provides a fixed endpoint.

- **`type: NodePort`** — Exposes the Service on a static port (`30007`) on **every node** in the cluster. Any external client can access the application via `<NodeIP>:30007`. This is ideal for development and testing.

- **`selector: app: nginx`** — The Service discovers which pods to route traffic to by matching this label. Any pod with `app: nginx` becomes a valid backend endpoint.

#### Port Mapping Breakdown

```
External Client → NodeIP:30007 → Service:80 → Pod:80
```

| Field | Value | Description |
|---|---|---|
| `nodePort` | `30007` | Port opened on every Node's IP for external access |
| `port` | `80` | Port the Service listens on within the cluster |
| `targetPort` | `80` | Port on the container that receives the forwarded traffic |

> ⚠️ **Note:** `NodePort` values must be in the range `30000–32767`. Using `NodePort` in production is uncommon — prefer `LoadBalancer` (cloud) or `Ingress` (advanced routing).

---

### 9. Traffic Flow

Here is the complete end-to-end path a request takes from a browser to the Nginx container:

```
┌─────────────────────────────────────────────────────┐
│                  External Client                    │
│              (Browser / curl / etc.)                │
└────────────────────────┬────────────────────────────┘
                         │  HTTP Request
                         ▼
┌─────────────────────────────────────────────────────┐
│             Kubernetes Node                         │
│                                                     │
│         <NodeIP>:30007  (NodePort)                  │
└────────────────────────┬────────────────────────────┘
                         │  Forwarded to Service
                         ▼
┌─────────────────────────────────────────────────────┐
│         Service: nginx-service (port 80)            │
│         Load balances across healthy pods           │
└────────────┬───────────────────────────┬────────────┘
             │                           │
             ▼                           ▼
┌────────────────────┐       ┌────────────────────────┐
│  Pod 1             │       │  Pod 2                 │
│  nginx (port 80)   │       │  nginx (port 80)       │
│  app: nginx ✅     │       │  app: nginx ✅         │
└────────────────────┘       └────────────────────────┘
```

The Service automatically load-balances traffic between both pods. If Pod 1 fails its readiness probe, all traffic is routed to Pod 2 until Pod 1 recovers.

---

## 📊 Reference Tables

### Complete Resource Summary

| Field | Value |
|---|---|
| Deployment Name | `nginx-deployment` |
| Container Image | `nginx:latest` |
| Replicas | `2` |
| Container Port | `80` |
| CPU Request / Limit | `100m` / `500m` |
| Memory Request / Limit | `128Mi` / `256Mi` |
| Liveness Check | `GET /` every 20s (starts at 15s) |
| Readiness Check | `GET /` every 10s (starts at 5s) |
| Service Name | `nginx-service` |
| Service Type | `NodePort` |
| External Port | `30007` |

---

## 📌 Notes

- **Pin your image version** — Using `nginx:latest` in production is risky as it may pull breaking changes. Prefer a specific tag like `nginx:1.25.3`.
- **NodePort vs LoadBalancer** — `NodePort` is great for local development. For cloud deployments, use `type: LoadBalancer` or an `Ingress` controller for proper SSL termination and routing.
- **Namespace isolation** — By default, resources are created in the `default` namespace. For production, use dedicated namespaces (`kubectl apply -f nginx-deployment.yaml -n my-namespace`).
- **Scaling** — To scale horizontally: `kubectl scale deployment nginx-deployment --replicas=5`

---

## 📜 License

This project is licensed under the [MIT License](LICENSE).
