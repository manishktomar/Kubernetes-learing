# Kubernetes Installation

This directory covers how to get a Kubernetes cluster running — locally for learning and on the three major cloud providers for production use.

---

## Files

| File | Description |
|---|---|
| [local-installation.yaml](local-installation.yaml) | Minikube, kind, k3s, k3d — local dev clusters |
| [eks.yaml](eks.yaml) | Amazon EKS — eksctl ClusterConfig, AWS CLI commands |
| [aks.yaml](aks.yaml) | Azure AKS — az CLI commands, ARM/Bicep reference |
| [gks.yaml](gks.yaml) | Google GKE — gcloud CLI, Config Connector CRD reference |

---

## Quick Comparison

| | Minikube | kind | k3s | EKS | AKS | GKE |
|---|---|---|---|---|---|---|
| **Type** | Local | Local (Docker) | Lightweight | Managed cloud | Managed cloud | Managed cloud |
| **Multi-node** | Yes (1.25+) | Yes | Yes | Yes | Yes | Yes |
| **Best for** | Learning | CI/CD & testing | Edge / IoT | AWS workloads | Azure workloads | GCP workloads |
| **Cost** | Free | Free | Free | Pay per node | Free control plane | Pay per node |
| **Autopilot** | No | No | No | No | No | Yes |

---

## Local Setup (Quick Start)

### Minikube
```bash
# Install
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Start
minikube start --cpus=2 --memory=4096

# Verify
kubectl get nodes
```

### kind (multi-node)
```bash
# Install
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.22.0/kind-linux-amd64
chmod +x ./kind && sudo mv ./kind /usr/local/bin/kind

# Create cluster using config in this directory
kind create cluster --config local-installation.yaml

# Verify
kubectl cluster-info --context kind-learning-cluster
```

### k3s (single command)
```bash
curl -sfL https://get.k3s.io | sh -
sudo kubectl get nodes
```

---

## Cloud Setup (Quick Start)

### AWS EKS
```bash
# Install eksctl, then:
eksctl create cluster -f eks.yaml
aws eks update-kubeconfig --region us-east-1 --name learning-cluster
kubectl get nodes
```

### Azure AKS
```bash
az group create --name k8s-learning-rg --location eastus
az aks create --resource-group k8s-learning-rg --name learning-cluster \
  --node-count 3 --generate-ssh-keys
az aks get-credentials --resource-group k8s-learning-rg --name learning-cluster
kubectl get nodes
```

### Google GKE
```bash
gcloud services enable container.googleapis.com
gcloud container clusters create-auto learning-cluster --region us-central1
gcloud container clusters get-credentials learning-cluster --region us-central1
kubectl get nodes
```

---

## Key Concepts

- **kubeconfig** — file at `~/.kube/config` that stores cluster credentials and contexts
- **context** — a named combination of cluster + user + namespace
- **kubectl** — the CLI for interacting with any Kubernetes cluster regardless of where it runs
- **CNI plugin** — Container Network Interface; handles pod networking (Calico, Cilium, Flannel, Azure CNI, etc.)
- **CSI driver** — Container Storage Interface; handles persistent storage (EBS, Azure Disk, GCE PD)
- **Managed node group / node pool** — a set of nodes managed as a unit (auto-repair, auto-upgrade)

---

## Context Management
```bash
kubectl config get-contexts                          # list all contexts
kubectl config current-context                       # show active context
kubectl config use-context kind-learning-cluster     # switch context
kubectl config set-context --current --namespace=dev # set default namespace
```
