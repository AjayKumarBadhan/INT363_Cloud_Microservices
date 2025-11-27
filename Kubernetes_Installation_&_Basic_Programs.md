# Kubernetes Installation & Command Reference Guide

A comprehensive guide for installing and managing Kubernetes on Windows using kubectl, Minikube, and Docker Desktop.

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Installing kubectl](#installing-kubectl)
- [Installing Minikube](#installing-minikube)
- [Deploying Applications](#deploying-applications)
  - [Program 1: NodePort Service](#program-1-nodeport-service)
  - [Program 2: LoadBalancer Service](#program-2-loadbalancer-service)
  - [Program 3: NGINX Deployment](#program-3-nginx-deployment)
- [Command Reference](#command-reference)
  - [Minikube Commands](#minikube-commands)
  - [Deployment & Pod Management](#deployment--pod-management)
  - [Service, Network & Ingress](#service-network--ingress)
  - [Configuration & Resources](#configuration--resources)
  - [Resource Monitoring](#resource-monitoring)
  - [Cleanup Commands](#cleanup-commands)
  - [YAML File Commands](#yaml-file-commands)
  - [Namespace Management](#namespace-management)
  - [Frequently Used Shortcuts](#frequently-used-shortcuts)

---

## Prerequisites

Before installing Kubernetes, ensure you have:

- Windows operating system
- Docker Desktop installed and running
- Administrator access to PowerShell
- Chocolatey package manager (will be verified during installation)

---

## Installing kubectl

kubectl is the command-line tool for interacting with Kubernetes clusters.

**Reference:** [Official kubectl Installation Guide](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/#install-nonstandard-package-tools)

### Step 1: Start Docker Desktop

Open **Docker Desktop** on your Windows system and keep it running in the background.

### Step 2: Open PowerShell as Administrator

Launch **PowerShell** in **administrator** mode by right-clicking and selecting "Run as Administrator".

### Step 3: Verify Chocolatey Installation

Check if Chocolatey package manager is installed:

```powershell
choco --version
```

If Chocolatey is not installed, visit [chocolatey.org](https://chocolatey.org/) for installation instructions.

### Step 4: Install kubectl

Install Kubernetes CLI using Chocolatey:

```powershell
choco install kubernetes-cli
```

This command downloads and installs the latest stable version of kubectl on your system.

### Step 5: Verify kubectl Installation

Test that kubectl is properly installed and check the version:

```powershell
kubectl version --client
```

This displays the client version of kubectl installed on your machine.

### Step 6: Check Cluster Configuration

Verify kubectl is properly configured by checking the cluster state:

```powershell
kubectl cluster-info
```

This command returns the URL of the Kubernetes control plane if properly configured.

### Step 7: Troubleshoot Connection (If Needed)

If you can't access your cluster despite getting a URL response, dump the cluster information for debugging:

```powershell
kubectl cluster-info dump
```

---

## Installing Minikube

Minikube runs a single-node Kubernetes cluster locally for development and testing.

**Reference:** [Official Minikube Installation Guide](https://minikube.sigs.k8s.io/docs/start/?arch=%2Fwindows%2Fx86-64%2Fstable%2Fchocolatey)

### Basic Requirements for Minikube

- 2 CPUs or more
- 2GB of free memory
- 20GB of free disk space
- Container or virtual machine manager (Docker, Hyper-V, VirtualBox, etc.)
- Internet connection

### Step 1: Install Minikube

Install the latest stable release of Minikube using Chocolatey:

```powershell
choco install minikube
```

### Step 2: Start the Cluster

Start Minikube with Docker as the driver:

```powershell
minikube start --driver=docker
```

**What this does:** Minikube creates a local Kubernetes cluster running inside a Docker container. The `--driver=docker` flag specifies that Docker should be used as the container runtime.

### Step 3: Interact with the Cluster

Verify the cluster is running by listing all pods across all namespaces:

```powershell
minikube kubectl -- get po -A
```

**Explanation:** This command uses Minikube's bundled kubectl to get all pods (`po`) across all namespaces (`-A`). You should see system pods running in namespaces like `kube-system`.

### Step 4: Access the Dashboard

Launch the Kubernetes web dashboard:

```powershell
minikube dashboard
```

This opens the Kubernetes dashboard in your default web browser, providing a graphical interface to manage your cluster.

### Step 5: Get Dashboard URL (Optional)

If you prefer not to open a browser automatically, get the dashboard URL:

```powershell
minikube dashboard --url
```

---

## Deploying Applications

### Program 1: NodePort Service

This example demonstrates creating a simple deployment and exposing it using a NodePort service.

#### Step 1: Create Deployment

Create a deployment named `hello-minikube` using the echo-server image:

```powershell
kubectl create deployment hello-minikube --image=kicbase/echo-server:1.0
```

**What this does:** Creates a deployment that runs a simple echo server container. The deployment manages a pod running the specified image.

#### Step 2: Expose Deployment

Expose the deployment on port 8080 using a NodePort service:

```powershell
kubectl expose deployment hello-minikube --type=NodePort --port=8080
```

**What this does:** Creates a service that exposes the deployment. `NodePort` makes the service accessible on a port on each node's IP. The `--port=8080` specifies the port the service listens on.

#### Step 3: Check Service Status

Verify the service was created successfully:

```powershell
kubectl get services hello-minikube
```

This displays the service details including the assigned NodePort.

#### Step 4: Access the Service

Open the service in your browser using Minikube's helper:

```powershell
minikube service hello-minikube
```

This automatically opens your default browser and navigates to the service.

#### Step 5: Get Service URL (Alternative)

If you only want the URL without opening a browser:

```powershell
minikube service hello-minikube --url
```

#### Step 6: Port Forwarding (Alternative Access Method)

Alternatively, use kubectl to forward the port to localhost:

```powershell
kubectl port-forward service/hello-minikube 7080:8080
```

**What this does:** Forwards local port 7080 to port 8080 on the service. Now you can access the application at `http://localhost:7080/`

Open your browser and navigate to: `http://localhost:7080/`

---

### Program 2: LoadBalancer Service

This example shows how to create a LoadBalancer service, which simulates cloud load balancers locally.

#### Step 1: Create Deployment

Create a deployment named `balanced`:

```powershell
kubectl create deployment balanced --image=kicbase/echo-server:1.0
```

#### Step 2: Expose as LoadBalancer

Expose the deployment using a LoadBalancer service type:

```powershell
kubectl expose deployment balanced --type=LoadBalancer --port=8080
```

**What this does:** Creates a LoadBalancer service. In cloud environments, this provisions a cloud load balancer. With Minikube, we need to create a tunnel to simulate this.

#### Step 3: Start Minikube Tunnel

In a separate PowerShell window, start the tunnel:

```powershell
minikube tunnel
```

**What this does:** Creates a network route on your host to services deployed with type LoadBalancer, providing them with an external IP address. Keep this terminal window open while using the LoadBalancer.

**Note:** You may need to provide administrator credentials when running this command.

#### Step 4: Get External IP

Find the routable IP address assigned to your service:

```powershell
kubectl get services balanced
```

Look for the `EXTERNAL-IP` column. This is the IP address you'll use to access your service.

#### Step 5: Access the Application

Your deployment is now available at `<EXTERNAL-IP>:8080`. Open your browser and navigate to this address.

---

### Program 3: NGINX Deployment

This example deploys an NGINX web server to your Kubernetes cluster.

#### Step 1: Create NGINX Deployment

Create a deployment running NGINX:

```powershell
kubectl create deployment nginx --image=nginx
```

**What this does:** Pulls the official NGINX image from Docker Hub and creates a deployment to manage NGINX pods.

#### Step 2: Expose NGINX (For Remote Access)

If running on a remote system, expose NGINX using NodePort:

```powershell
kubectl expose deployment nginx --port=80 --type=NodePort
```

**What this does:** Exposes NGINX on port 80 and makes it accessible via a NodePort on your cluster nodes.

#### Step 3: Check Resources

Verify the pods are running:

```powershell
kubectl get pods
```

Check the deployment status:

```powershell
kubectl get deployment
```

Both commands help you verify that NGINX is running correctly. You should see the nginx pod in `Running` state.

#### Step 4: Get Service URL

Retrieve the URL to access NGINX:

```powershell
minikube service nginx --url
```

This returns the URL where your NGINX server is accessible.

#### Step 5: Access NGINX

Open your browser and navigate to the URL provided in Step 4. You should see the default NGINX welcome page.

---

## Command Reference

### Minikube Commands

| Command | Description |
|---------|-------------|
| `minikube start` | Start Minikube cluster |
| `minikube stop` | Stop the cluster |
| `minikube delete` | Delete the cluster |
| `minikube status` | Show status of cluster |
| `minikube dashboard` | Launch Kubernetes dashboard |
| `minikube service <svc-name>` | Open exposed service in browser |
| `minikube ip` | Get Minikube VM IP |
| `minikube ssh` | SSH into Minikube node |
| `minikube addons list` | List available addons |
| `minikube addons enable ingress` | Enable Ingress controller |
| `minikube addons enable metrics-server` | Enable metrics server for HPA |
| `minikube logs` | View Minikube logs |
| `eval $(minikube docker-env)` | Use local Docker daemon |
| `minikube tunnel` | Enable LoadBalancer access (cloud-like) |

---

### Deployment & Pod Management

| Command | Description |
|---------|-------------|
| `kubectl get pods` | List pods |
| `kubectl get deployment` | List deployments |
| `kubectl get rs` | List replica sets |
| `kubectl get all` | Show all resources |
| `kubectl get pod -w` | Watch pod changes |
| `kubectl describe pod <pod-name>` | Detailed pod info |
| `kubectl logs <pod-name>` | Get logs of pod |
| `kubectl logs -f <pod-name>` | Follow live logs |
| `kubectl exec -it <pod-name> -- /bin/bash` | Access shell inside pod |
| `kubectl scale deployment <name> --replicas=5` | Scale pods |
| `kubectl edit deployment <name>` | Edit live deployment |
| `kubectl rollout status deployment <name>` | Check rollout status |
| `kubectl rollout undo deployment <name>` | Rollback previous version |
| `kubectl delete pod <pod-name>` | Delete pod |
| `kubectl delete deployment <name>` | Delete deployment |

---

### Service, Network & Ingress

| Command | Description |
|---------|-------------|
| `kubectl get svc` | List services |
| `kubectl expose deployment <name> --type=NodePort --port=80` | Expose deployment |
| `kubectl get endpoints` | List service endpoints |
| `kubectl get ingress` | List ingress |
| `kubectl describe svc <name>` | Detailed service info |
| `kubectl port-forward svc/<name> 8080:80` | Forward port |

---

### Configuration & Resources

| Command | Description |
|---------|-------------|
| `kubectl get configmap` | List ConfigMaps |
| `kubectl describe configmap <name>` | Detailed ConfigMap info |
| `kubectl get secret` | List Secrets |
| `kubectl describe secret <name>` | Show secret info |
| `kubectl config view` | View kubeconfig |
| `kubectl config get-contexts` | Show contexts |
| `kubectl config use-context <context-name>` | Switch context |

---

### Resource Monitoring

| Command | Description |
|---------|-------------|
| `kubectl top pod` | Show pod resource usage |
| `kubectl top node` | Show node resource usage |
| `kubectl get events --sort-by=.metadata.creationTimestamp` | Show recent events |
| `kubectl get nodes -o wide` | Show node IPs & details |

---

### Cleanup Commands

| Command | Description |
|---------|-------------|
| `kubectl delete all -l app=<name>` | Delete using label |
| `kubectl delete deploy <name>` | Delete deployment |
| `kubectl delete svc <name>` | Delete service |
| `kubectl delete pod <name>` | Delete pod |
| `kubectl delete namespace <name>` | Delete namespace |
| `kubectl delete all --all` | Delete everything in the namespace (⚠️ use carefully) |

---

### YAML File Commands

| Command | Description |
|---------|-------------|
| `kubectl apply -f file.yaml` | Apply resources |
| `kubectl delete -f file.yaml` | Delete resources |
| `kubectl replace -f file.yaml --force --grace-period=0` | Force applies |
| `kubectl diff -f file.yaml` | Show differences before apply |
| `kubectl create -f file.yaml` | Create if not exist |

---

### Namespace Management

| Command | Description |
|---------|-------------|
| `kubectl get namespace` | Show namespaces |
| `kubectl create namespace dev` | Create namespace |
| `kubectl delete namespace test` | Delete namespace |
| `kubectl config set-context --current --namespace=dev` | Switch namespace |

---

### Frequently Used Shortcuts

```powershell
kubectl get po        # Pods
kubectl get deploy    # Deployments
kubectl get svc       # Services
kubectl get ns        # Namespaces
kubectl get no        # Nodes
```

---

## Additional Tips

- Always keep Docker Desktop running when working with Minikube
- Use `kubectl get all` to quickly view all resources in your current namespace
- The `minikube tunnel` command requires administrator privileges and must stay running
- Use `kubectl describe` commands when troubleshooting issues with resources
- The `-w` flag (watch) is useful for monitoring real-time changes to resources

---

## Troubleshooting

**Issue:** Minikube fails to start
- Ensure Docker Desktop is running
- Try `minikube delete` then `minikube start` to recreate the cluster

**Issue:** Cannot access services
- Verify the service is running: `kubectl get svc`
- Check pod status: `kubectl get pods`
- Use `kubectl logs <pod-name>` to check for application errors

**Issue:** LoadBalancer pending
- Ensure `minikube tunnel` is running in a separate terminal
- Check if you have administrator privileges

---

## Contributing

Feel free to submit issues or pull requests if you find any errors or have suggestions for improvements.

## License

This guide is provided as-is for educational purposes.
