# 🚀 Nginx Kubernetes Deployment — Minikube Dashboard

A guide to deploying an Nginx web server on Kubernetes using the **Minikube Dashboard** — a browser-based UI to manage your cluster without using the command line.

---

## 📋 Table of Contents

- [Full Configuration Code](#-full-configuration-code)
- [Prerequisites](#-prerequisites)
- [Implementation via Minikube Dashboard](#-implementation-via-minikube-dashboard)
  - [Step 1 — Start Minikube and Launch Dashboard](#step-1--start-minikube-and-launch-the-dashboard)
  - [Step 2 — Deploy the Configuration](#step-2--deploy-the-configuration)
  - [Step 3 — Verify the Deployment](#step-3--verify-the-deployment)
  - [Step 4 — Access the Application](#step-4--access-the-application)
- [Detailed Configuration Explanation](#-detailed-configuration-explanation)
  - [Deployment Resource](#1-deployment-resource)
  - [Metadata and Labels](#2-metadata-and-labels)
  - [Replicas and Selector](#3-replicas-and-selector)
  - [Pod Template](#4-pod-template)
  - [Resource Requests and Limits](#5-resource-requests-and-limits)
  - [Liveness Probe](#6-liveness-probe)
  - [Readiness Probe](#7-readiness-probe)
  - [Service Resource](#8-service-resource)
  - [Traffic Flow](#9-traffic-flow)
- [Notes](#-notes)
- [License](#-license)

---

## 📄 Full Configuration Code

Save this file locally as `nginx-deployment.yaml` before proceeding.

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

## ✅ Prerequisites

- [Minikube](https://minikube.sigs.k8s.io/docs/start/) installed and running
- [kubectl](https://kubernetes.io/docs/tasks/tools/) installed
- `nginx-deployment.yaml` saved locally

---

## 🖥️ Implementation via Minikube Dashboard

### Step 1 — Start Minikube and Launch the Dashboard

Start your Minikube cluster:

```bash
minikube start
```

Launch the Kubernetes Dashboard:

```bash
minikube dashboard
```

> ✅ This opens the Dashboard automatically in your default browser. No login or token is required.

---

### Step 2 — Deploy the Configuration

Once inside the Dashboard:

1. Click the **`+`** (Create) button in the **top-right corner**
2. Select the **"Upload from file"** tab
3. Click **"Browse"** and select your `nginx-deployment.yaml` file
4. Click **"Upload"**

The Dashboard will apply both the `Deployment` and `Service` resources from the file at once.

> 💡 Alternatively, select the **"Input"** tab, paste the YAML directly into the editor, and click **"Upload"**.

---

### Step 3 — Verify the Deployment

After uploading, confirm everything is running correctly inside the Dashboard:

| What to check | Where in Dashboard | Expected Result |
|---|---|---|
| Pod status | Workloads → Pods | 2 pods with ✅ green status |
| Deployment health | Workloads → Deployments | `nginx-deployment` shows **2/2** ready |
| Service created | Discovery and Load Balancing → Services | `nginx-service` with NodePort `30007` |
| Container logs | Pods → (pod name) → Logs icon | Nginx access logs streaming live |

---

### Step 4 — Access the Application

Run the following command to get the accessible URL for the service:

```bash
minikube service nginx-service --url
```

Open the returned URL in your browser. You should see the **Nginx welcome page**.

Alternatively, access it directly via:

```
http://<minikube-ip>:30007
```

Get your Minikube IP with:

```bash
minikube ip
```

---

## 🔍 Detailed Configuration Explanation

### 1. Deployment Resource

```yaml
apiVersion: apps/v1
kind: Deployment
```

- **`apiVersion: apps/v1`** — Specifies the stable Kubernetes API group for workload resources. Required for all Deployment objects in modern clusters.
- **`kind: Deployment`** — Instructs Kubernetes to create a Deployment controller that watches the cluster and ensures the desired number of Nginx pods are always running. Visible under **Workloads → Deployments** in the Dashboard.

---

### 2. Metadata and Labels

```yaml
metadata:
  name: nginx-deployment
  labels:
    app: nginx
```

- **`name: nginx-deployment`** — The unique name of this Deployment within the cluster. This is the name you will see listed in the Dashboard's Deployments view.
- **`labels: app: nginx`** — A key-value tag attached to the resource. Labels are used for grouping, filtering, and selecting resources. The `app: nginx` label is used by the Service to discover and route traffic to the correct pods.

---

### 3. Replicas and Selector

```yaml
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
```

- **`replicas: 2`** — Kubernetes maintains exactly 2 running Nginx pods at all times. If a pod crashes or is deleted, a new one is automatically scheduled. The Dashboard shows this as **2/2** in the Deployments view.
- **`selector.matchLabels: app: nginx`** — Tells the Deployment which pods it is responsible for. Only pods carrying the label `app: nginx` are managed by this Deployment.

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

- **`template`** — The blueprint Kubernetes uses every time it needs to create a new pod replica.
- **`labels: app: nginx`** — Every pod created from this template receives this label, making it discoverable by the Deployment selector and the Service.
- **`image: nginx:latest`** — Pulls the official Nginx Docker image. The image name is visible on the pod detail page in the Dashboard.
- **`containerPort: 80`** — Declares that the Nginx container listens on port 80 (HTTP). Shown in the Dashboard under the container's port information.

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

**Requests** are the minimum resources guaranteed for scheduling. **Limits** are the maximum a container can consume before being throttled or restarted.

| Type | Resource | Value | Behaviour |
|---|---|---|---|
| Request | CPU | `100m` (0.1 core) | Guaranteed allocation on the node |
| Request | Memory | `128Mi` | Guaranteed RAM on the node |
| Limit | CPU | `500m` (0.5 core) | Container is throttled if exceeded |
| Limit | Memory | `256Mi` | Container is **OOMKilled** and restarted if exceeded |

> 💡 With `metrics-server` enabled (`minikube addons enable metrics-server`), the Dashboard shows live CPU and memory usage graphs per pod against these limits.

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

The liveness probe answers: **"Is this container still alive and functional?"**

Kubernetes sends an HTTP GET to `http://localhost:80/` inside the container on the following schedule:

| Parameter | Value | Meaning |
|---|---|---|
| `initialDelaySeconds` | `15` | Wait 15 seconds after startup before the first check |
| `periodSeconds` | `20` | Repeat the check every 20 seconds |

**On failure:** After 3 consecutive failures, Kubernetes **restarts** the container. The Dashboard shows an increasing **Restart Count** on the affected pod.

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

The readiness probe answers: **"Is this container ready to serve traffic?"**

| Parameter | Value | Meaning |
|---|---|---|
| `initialDelaySeconds` | `5` | Begin checking 5 seconds after startup |
| `periodSeconds` | `10` | Repeat the check every 10 seconds |

**On failure:** The pod is marked **Not Ready** and removed from the Service's load balancer until it recovers. The Dashboard highlights this with a yellow/orange pod indicator.

> 🔑 **Key Difference Between Probes**

| Probe | On Failure | Dashboard Indicator |
|---|---|---|
| **Liveness** | Container is restarted | Restart count increases |
| **Readiness** | Traffic is stopped to the pod | Pod status shows "Not Ready" |

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

A **Service** provides a stable, fixed network endpoint for accessing pods, whose IPs change on every restart.

- **`type: NodePort`** — Exposes the Service on a static port (`30007`) on the Minikube node, making it reachable from outside the cluster.
- **`selector: app: nginx`** — Automatically routes traffic to any pod labeled `app: nginx`. The Dashboard shows the matched pods under **Endpoints** in the service detail view.

#### Port Mapping Breakdown

```
External Browser → <minikube-ip>:30007 → Service port 80 → Container port 80
```

| Field | Value | Description |
|---|---|---|
| `nodePort` | `30007` | External port exposed on the Minikube node |
| `port` | `80` | Port the Service listens on inside the cluster |
| `targetPort` | `80` | Port on the Nginx container receiving the traffic |

---

### 9. Traffic Flow

```
┌─────────────────────────────────────────────────┐
│              External Browser                   │
└────────────────────┬────────────────────────────┘
                     │  HTTP Request
                     ▼
┌─────────────────────────────────────────────────┐
│             Minikube Node                       │
│          <minikube-ip>:30007                    │
└────────────────────┬────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────┐
│      Service: nginx-service (port 80)           │
│      Load balances across healthy pods          │
└──────────┬──────────────────────────┬───────────┘
           │                          │
           ▼                          ▼
┌─────────────────────┐    ┌─────────────────────┐
│  Pod 1              │    │  Pod 2              │
│  nginx (port 80)    │    │  nginx (port 80)    │
│  ✅ Ready           │    │  ✅ Ready           │
└─────────────────────┘    └─────────────────────┘
```

Traffic is load-balanced between both pods. Only pods that pass the readiness probe receive traffic.

---

## 📌 Notes

- **Pin your image version** — Replace `nginx:latest` with a specific tag like `nginx:1.25.3` in production to prevent unexpected breaking changes.
- **NodePort range** — NodePort values must fall between `30000–32767`. The value `30007` used here is valid.
- **Enable metrics-server** — Run `minikube addons enable metrics-server` to see live CPU and memory charts in the Dashboard.
- **Scale from the Dashboard** — Navigate to **Workloads → Deployments → nginx-deployment** and use the scale button to adjust replicas without editing any files.

---

## 📜 License

This project is licensed under the [MIT License](LICENSE).
