# 🚀 ArgoCD Setup & Installation Guide

Easily set up **ArgoCD (UI + CLI)** on a local or cloud-based Kubernetes cluster using **Kind** and **Helm**.  
This guide includes all required tools, installation steps, and commands to access the ArgoCD dashboard from your browser.

---

## 🧩 Prerequisites

Make sure you have the following installed on your system:

| Tool | Description | Install / Verify |
|------|--------------|-----------------|
| 🐳 **Docker** | Required for Kind to run cluster nodes as containers. | ```bash sudo apt-get update && sudo apt install docker.io -y && sudo usermod -aG docker $USER && newgrp docker docker --version docker ps ``` |
| ☸️ **Kind (Kubernetes in Docker)** | To create lightweight Kubernetes clusters. | ```bash kind version ``` |
| ⚙️ **kubectl** | To interact with your cluster. | ```bash kubectl version --client ``` |
| 🧰 **Helm** | For Helm-based ArgoCD installation. | ```bash helm version ``` |

---

## ⚠️ Important Note
You can **either** follow the manual steps below  
**or** simply execute the provided script:

----

🏗️ Step 1: Create Kind Cluster

Create your kind-config.yaml file:

```bash
bash setup_argocd.sh
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  apiServerAddress: "172.31.19.178"  # ⚠️ Replace with your EC2 private IP
  apiServerPort: 33893
nodes:
  - role: control-plane
    image: kindest/node:v1.33.1
  - role: worker
    image: kindest/node:v1.33.1
  - role: worker
    image: kindest/node:v1.33.1
```
----


Create and verify your cluster:

```bash
kind create cluster --name argocd-cluster --config kind-config.yaml
kubectl cluster-info
kubectl get nodes

```

Step 2: Install ArgoCD
💎 Method 1 — Install via Helm (recommended)

```bash
# Add Argo Helm repo
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update

# Create namespace
kubectl create namespace argocd

# Install ArgoCD
helm install argocd argo/argo-cd -n argocd

# Verify installation
kubectl get pods -n argocd
kubectl get svc -n argocd


```

🌐 Access the UI:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443 --address=0.0.0.0 &

```

🔐 Get initial credentials:

```bash
kubectl get secret argocd-initial-admin-secret -n argocd \
  -o jsonpath="{.data.password}" | base64 -d && echo

```
Login:

Username → admin

Password → (above output)

Method 2 — Install via Official Manifests

```bash
# Create namespace
kubectl create namespace argocd

# Apply official installation manifest
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Verify installation
kubectl get pods -n argocd
kubectl get svc -n argocd

```
💻 Step 3: Install ArgoCD CLI (Linux/Ubuntu)

To manage ArgoCD from the terminal, install the CLI tool:

```bash
curl -sSL -o argocd-linux-amd64 \
  https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64

sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64

```
✅ Verify installation:

```bash
argocd version --client

```
🔐 Login to CLI:

```bash
argocd login <public_ip>:8080 --username admin --password <password> --insecure

```
Feel free to star this repo if you found it useful!
🚀 Happy Deploying with ArgoCD!
