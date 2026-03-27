# 🚀 Nginx Kubernetes Deployment via Kubernetes Dashboard

A step-by-step guide to deploying an Nginx web server on Kubernetes using the **Kubernetes Dashboard** — a web-based UI for managing your cluster — instead of the command line. This covers installing the dashboard, accessing it, and deploying the full configuration through the UI.

---

## 📋 Table of Contents

- [Full Configuration Code](#-full-configuration-code)
- [Overview](#-overview)
- [Prerequisites](#-prerequisites)
- [Step 1 — Install the Kubernetes Dashboard](#-step-1--install-the-kubernetes-dashboard)
- [Step 2 — Create an Admin Service Account](#-step-2--create-an-admin-service-account)
- [Step 3 — Access the Dashboard](#-step-3--access-the-dashboard)
- [Step 4 — Deploy Nginx via the Dashboard](#-step-4--deploy-nginx-via-the-dashboard)
- [Step 5 — Expose Nginx with a NodePort Service](#-step-5--expose-nginx-with-a-nodeport-service)
- [Step 6 — Verify and Monitor via the Dashboard](#-step-6--verify-and-monitor-via-the-dashboard)
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
- [Reference Tables](#-reference-tables)
- [Troubleshooting](#-troubleshooting)
- [Notes](#-notes)
- [License](#-license)

---

## 📄 Full Configuration Code

Save this as `nginx-deployment.yaml`. This file will be uploaded directly into the Kubernetes Dashboard UI.

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

| Resource | Name | Purpose |
|---|---|---|
| `Deployment` | `nginx-deployment` | Runs 2 replicas of Nginx with health probes and resource limits |
| `Service` | `nginx-service` | Exposes Nginx externally via NodePort `30007` |

The **Kubernetes Dashboard** provides a graphical alternative to `kubectl`. You can upload YAML files, monitor pods, inspect logs, and manage resources — all from a browser.

---

## ✅ Prerequisites

- A running Kubernetes cluster:
  - Local: [Minikube](https://minikube.sigs.k8s.io/) or [Kind](https://kind.sigs.k8s.io/)
  - Cloud: GKE, EKS, or AKS
- [`kubectl`](https://kubernetes.io/docs/tasks/tools/) installed and configured
- The `nginx-deployment.yaml` file saved locally (use the code above)

---

## 🖥️ Step 1 — Install the Kubernetes Dashboard

### Option A: Minikube (Recommended for Local)

Minikube ships with the Dashboard as a built-in addon. Enable it with a single command:

```bash
minikube addons enable dashboard
minikube addons enable metrics-server
```

Then launch it:

```bash
minikube dashboard
```

> ✅ This automatically opens the Dashboard in your browser. No token login is required for Minikube.

---

### Option B: Standard Kubernetes Cluster

Apply the official Dashboard manifest:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

Verify the Dashboard pods are running:

```bash
kubectl get pods -n kubernetes-dashboard
```

Expected output:

```
NAME                                         READY   STATUS    RESTARTS   AGE
dashboard-metrics-scraper-xxx                1/1     Running   0          1m
kubernetes-dashboard-xxx                     1/1     Running   0          1m
```

---

## 🔐 Step 2 — Create an Admin Service Account

> ⚠️ **Skip this step if using Minikube** — it handles authentication automatically.

For standard clusters, you must create a service account with admin access and generate a login token.

### 2a. Create the Service Account

Save the following as `dashboard-admin.yaml`:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```

Apply it:

```bash
kubectl apply -f dashboard-admin.yaml
```

### 2b. Generate the Login Token

```bash
kubectl -n kubernetes-dashboard create token admin-user
```

Copy the token output — you will paste it into the Dashboard login screen.

---

## 🌐 Step 3 — Access the Dashboard

### On Minikube

```bash
minikube dashboard
```

The browser opens automatically at the Dashboard URL.

---

### On Standard Clusters

Start the proxy to forward the Dashboard to your local machine:

```bash
kubectl proxy
```

Then open the following URL in your browser:

```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

On the login screen:

1. Select **"Token"**
2. Paste the token generated in Step 2b
3. Click **"Sign In"**

> 🔒 The Dashboard is only accessible via `localhost` when using `kubectl proxy`. It is not exposed to the internet.

---

## 📦 Step 4 — Deploy Nginx via the Dashboard

Once inside the Dashboard, follow these steps to deploy the Nginx configuration using your YAML file.

### Method A: Upload the YAML File (Recommended)

1. Click the **`+`** (Create) button in the top-right corner of the Dashboard
2. Select the **"Upload from file"** tab
3. Click **"Browse"** and select your `nginx-deployment.yaml` file
4. Click **"Upload"**

The Dashboard will parse and apply both the `Deployment` and `Service` resources defined in the file simultaneously.

---

### Method B: Paste YAML Directly

1. Click the **`+`** (Create) button
2. Select the **"Input"** tab (or "Create from input")
3. Paste the entire YAML content into the editor
4. Click **"Upload"**

---

### Method C: Use the Form-Based UI (Basic Only)

> ⚠️ The form UI does not support advanced fields like probes or resource limits. Use Methods A or B for the full configuration.

1. Click **`+`** → **"Create from form"**
2. Fill in:
   - **App name:** `nginx`
   - **Container image:** `nginx:latest`
   - **Number of pods:** `2`
   - **Service:** External, Port `80`, Target port `80`
3. Click **"Deploy"**

---

## 🔌 Step 5 — Expose Nginx with a NodePort Service

The `nginx-service` is already included in the YAML file uploaded in Step 4. To confirm the Service was created:

1. In the left sidebar, click **"Services"** under the **"Discovery and Load Balancing"** section
2. Look for **`nginx-service`** in the list
3. Confirm:
   - **Type:** `NodePort`
   - **Cluster IP:** assigned automatically
   - **External Endpoints:** shows port `30007`

### Access Your Application

```bash
# On Minikube — get the access URL automatically
minikube service nginx-service --url

# On standard clusters — get your Node IP
kubectl get nodes -o wide

# Then visit:
http://<NODE_IP>:30007
```

---

## 📊 Step 6 — Verify and Monitor via the Dashboard

The Dashboard gives you real-time visibility into your deployment. Here is what to check after deploying:

### Check Pod Status

1. Click **"Pods"** in the left sidebar
2. Both `nginx-deployment-xxxx` pods should show a ✅ **green status**
3. Click on any pod to see:
   - **Events** — scheduling and startup logs
   - **Logs** — live container stdout output
   - **Resource usage** — CPU and memory consumption (requires metrics-server)

### Check Deployment Health

1. Click **"Deployments"** in the sidebar
2. `nginx-deployment` should show **2/2** pods ready
3. The green bar indicates all replicas are healthy

### Check Service Endpoints

1. Click **"Services"** → `nginx-service`
2. Verify the **Endpoints** section lists the IPs of both running pods

### View Live Logs

1. Go to **"Pods"** → click a pod name
2. Click the **📄 Logs** icon (top-right of the pod detail page)
3. Nginx access logs appear in real time as requests come in

### Scale the Deployment from the Dashboard

1. Go to **"Deployments"** → `nginx-deployment`
2. Click the **✏️ Edit** (pencil) icon or the scale button
3. Change `replicas: 2` to any desired number
4. Click **"Update"** — the cluster adjusts immediately

---

## 🔍 Detailed Configuration Explanation

### 1. Deployment Resource

```yaml
apiVersion: apps/v1
kind: Deployment
```

- **`apiVersion: apps/v1`** — The stable Kubernetes API group for workload controllers. This version is required for Deployments in all modern clusters.
- **`kind: Deployment`** — Creates a Deployment controller that continuously reconciles the cluster state, ensuring the declared number of pod replicas is always running.

---

### 2. Metadata and Labels

```yaml
metadata:
  name: nginx-deployment
  labels:
    app: nginx
```

- **`name: nginx-deployment`** — Unique identifier for this resource. Visible in the Dashboard under **Deployments**.
- **`labels: app: nginx`** — A key-value tag. The Dashboard uses labels for filtering and grouping. The Service also uses this label to discover which pods to send traffic to.

---

### 3. Replicas and Selector

```yaml
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
```

- **`replicas: 2`** — Maintains exactly 2 running pods at all times. Visible as **"2/2 Running"** in the Dashboard's Deployments view. If a pod is deleted, Kubernetes immediately creates a replacement.
- **`selector.matchLabels`** — The Deployment tracks only pods with `app: nginx`. This decouples the controller from the pods it manages, allowing rolling updates and canary deployments.

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

- **`template`** — The pod blueprint used every time Kubernetes needs to create a new replica.
- **`labels: app: nginx`** — Pods get this label automatically, making them discoverable by both the Deployment selector and the Service selector.
- **`image: nginx:latest`** — Pulls the official Nginx image from Docker Hub. Visible in the Dashboard pod detail page.
- **`containerPort: 80`** — Declares the port Nginx listens on. The Dashboard shows this under the container's port information.

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

#### Requests — Scheduling Guarantee

| Resource | Value | Meaning |
|---|---|---|
| `cpu` | `100m` | 0.1 CPU core reserved for this pod on its node |
| `memory` | `128Mi` | 128 MiB of RAM reserved for this pod |

#### Limits — Runtime Cap

| Resource | Value | Behaviour when exceeded |
|---|---|---|
| `cpu` | `500m` | Container is CPU-throttled (slowed down, not killed) |
| `memory` | `256Mi` | Container is **OOMKilled** and restarted automatically |

> 💡 In the Dashboard, go to **Pods → (pod name)** to see real-time CPU and memory graphs against these limits (requires metrics-server).

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

Answers: **"Is this container still alive?"**

Kubernetes sends `GET http://localhost:80/` inside the container every 20 seconds (after a 15-second startup grace period).

| Parameter | Value | Purpose |
|---|---|---|
| `initialDelaySeconds` | `15` | Gives Nginx time to start before the first check |
| `periodSeconds` | `20` | Frequency of health checks |

**On failure:** After 3 consecutive failures (default), Kubernetes **restarts** the container. This is visible in the Dashboard as an increased **Restart Count** on the pod.

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

Answers: **"Is this container ready to receive traffic?"**

| Parameter | Value | Purpose |
|---|---|---|
| `initialDelaySeconds` | `5` | Start checking 5 seconds after startup |
| `periodSeconds` | `10` | Recheck every 10 seconds |

**On failure:** The pod is marked **Not Ready** and temporarily removed from the Service's endpoint list. No traffic is sent to it. The Dashboard shows this as a yellow/orange pod status.

> 🔑 **Probe Comparison**

| Probe | On Failure | Dashboard Indicator |
|---|---|---|
| **Liveness** | Container restarted | Restart count increases |
| **Readiness** | Traffic removed | Pod shows "Not Ready" |

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

A **Service** provides a stable network endpoint for pods, whose IPs change on every restart.

- **`type: NodePort`** — Opens port `30007` on every node in the cluster for external access. Visible in the Dashboard under **Services → nginx-service → External Endpoints**.
- **`selector: app: nginx`** — Dynamically discovers all pods labeled `app: nginx` as traffic backends. The Dashboard shows these under **Endpoints** in the service detail view.

#### Port Mapping

| Field | Value | Description |
|---|---|---|
| `nodePort` | `30007` | External port on the Node IP |
| `port` | `80` | Internal cluster port of the Service |
| `targetPort` | `80` | Port on the Nginx container |

---

### 9. Traffic Flow

```
┌──────────────────────────────────────────────────────────┐
│                    External Client                       │
│                (Browser / curl / etc.)                   │
└───────────────────────────┬──────────────────────────────┘
                            │  HTTP Request
                            ▼
┌──────────────────────────────────────────────────────────┐
│                  Kubernetes Node                         │
│              <NodeIP>:30007  (NodePort)                  │
└───────────────────────────┬──────────────────────────────┘
                            │
                            ▼
┌──────────────────────────────────────────────────────────┐
│          Service: nginx-service (port 80)                │
│        Load balances across all ready pods               │
└──────────┬───────────────────────────────┬───────────────┘
           │                               │
           ▼                               ▼
┌──────────────────────┐       ┌───────────────────────────┐
│  Pod 1               │       │  Pod 2                    │
│  nginx (port 80)     │       │  nginx (port 80)          │
│  ✅ Ready            │       │  ✅ Ready                 │
└──────────────────────┘       └───────────────────────────┘
```

The Service only routes to pods that have **passed** their readiness probe. Both pods are visible and clickable in the Dashboard's **Pods** view.

---

## 📊 Reference Tables

### Full Configuration Summary

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

### Dashboard Navigation Cheatsheet

| Task | Dashboard Path |
|---|---|
| View all pods | Workloads → Pods |
| View deployments | Workloads → Deployments |
| View services | Discovery and Load Balancing → Services |
| View live logs | Pods → (pod name) → Logs icon |
| Scale replicas | Deployments → (name) → Scale icon |
| Edit YAML live | Any resource → Edit (pencil) icon |
| Delete a resource | Any resource → Delete (trash) icon |

---

## 🛠️ Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| Pods stuck in `Pending` | Insufficient node resources | Check node capacity: `kubectl describe nodes` |
| Pods in `CrashLoopBackOff` | Container failing to start | Check logs in Dashboard → Pods → Logs |
| Pods show `0/2 Ready` | Readiness probe failing | Check probe path and port match Nginx config |
| Dashboard login fails | Token expired or incorrect | Re-run `kubectl -n kubernetes-dashboard create token admin-user` |
| Cannot access `:30007` | Minikube networking issue | Use `minikube service nginx-service --url` instead |
| `ImagePullBackOff` | Cannot pull `nginx:latest` | Check internet/Docker Hub access from node |

---

## 📌 Notes

- **Pin your image version** — Replace `nginx:latest` with a specific version like `nginx:1.25.3` to avoid unexpected changes in production.
- **NodePort range** — NodePort values must be between `30000–32767`. The default `30007` is within this range.
- **Metrics-server required** — CPU/memory graphs in the Dashboard only appear if `metrics-server` is installed (`minikube addons enable metrics-server`).
- **Production alternatives** — For production, replace `NodePort` with `type: LoadBalancer` (cloud) or use an `Ingress` controller for SSL and host-based routing.
- **Scaling from Dashboard** — You can scale replicas live from the Dashboard without editing any files: Deployments → `nginx-deployment` → Scale.

---

## 📜 License

This project is licensed under the [MIT License](LICENSE).
