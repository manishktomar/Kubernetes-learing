# Kubernetes Cluster Architecture — Learning Reference

A Kubernetes cluster consists of three main layers:

1. **Control Plane** – the brain; manages cluster state
2. **Worker Nodes** – where your workloads actually run
3. **Add-ons** – optional but common cluster features

---

## Table of Contents

- [Control Plane Components](#control-plane-components)
  - [kube-apiserver](#kube-apiserver)
  - [etcd](#etcd)
  - [kube-scheduler](#kube-scheduler)
  - [kube-controller-manager](#kube-controller-manager)
  - [cloud-controller-manager](#cloud-controller-manager)
- [Node (Worker) Components](#node-worker-components)
  - [kubelet](#kubelet)
  - [kube-proxy](#kube-proxy)
  - [Container Runtime](#container-runtime)
- [Add-ons](#add-ons)
- [Cluster Communication Overview](#cluster-communication-overview)
- [Useful kubectl Commands](#useful-kubectl-commands)

---

## Control Plane Components

The control plane makes global decisions about the cluster (scheduling, detecting and responding to cluster events). It typically runs on dedicated master nodes — 3 for high availability.

### kube-apiserver

The front-end of the Kubernetes control plane. Exposes the Kubernetes REST API. All communication — `kubectl`, controllers, nodes — goes through it. Horizontally scalable; run multiple instances for HA.

**Key responsibilities:**
- Authentication & Authorization (RBAC)
- Admission control (validation, mutation)
- Storing state in etcd

**Port:** `6443` (HTTPS)

```yaml
image: registry.k8s.io/kube-apiserver:v1.29.0
flags:
  - "--etcd-servers=https://127.0.0.1:2379"
  - "--service-cluster-ip-range=10.96.0.0/16"
  - "--authorization-mode=Node,RBAC"
  - "--enable-admission-plugins=NodeRestriction"
```

---

### etcd

A consistent and highly-available key-value store. The single source of truth for all cluster state. Only the API server talks to etcd directly.

**Key facts:**
- Uses the Raft consensus algorithm (majority quorum required)
- Recommended: 3 or 5 etcd members for HA
- **Back up etcd regularly — losing it means losing all cluster state**

**Ports:** `2379` (client), `2380` (peer)

```bash
# Backup command
etcdctl snapshot save /backup/etcd-snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt \
  --key=/etc/kubernetes/pki/etcd/healthcheck-client.key
```

---

### kube-scheduler

Watches for newly created Pods with no assigned node and selects a node for them to run on.

**Scheduling factors:**
- Resource requirements (requests/limits)
- Node affinity / anti-affinity
- Taints & tolerations
- Pod topology spread constraints
- Node selector
- Available resources on nodes

**Port:** `10259` (HTTPS)

---

### kube-controller-manager

Runs controller loops that regulate the state of the cluster. Each controller is a separate goroutine compiled into one binary.

**Built-in controllers:**

| Controller | Responsibility |
|---|---|
| Node controller | Notices and responds to node failures |
| Job controller | Watches Job objects, creates pods |
| Endpoint controller | Links Services and Pods |
| ServiceAccount controller | Creates default SAs for namespaces |
| ReplicaSet controller | Maintains correct number of pod replicas |
| Deployment controller | Manages ReplicaSet rollovers |

**Port:** `10257` (HTTPS)

---

### cloud-controller-manager

Separates cloud-specific control logic from core Kubernetes. Only runs if the cluster is on a cloud provider.

**Cloud controllers:**
- **Node controller** – checks cloud API for deleted node instances
- **Route controller** – sets up cloud network routes
- **Service controller** – creates/updates/deletes cloud load balancers

**Provider implementations:**

| Provider | Image |
|---|---|
| AWS | `amazon/aws-cloud-controller-manager` |
| Azure | `mcr.microsoft.com/oss/kubernetes/azure-cloud-controller-manager` |
| GKE | Managed by Google |

---

## Node (Worker) Components

Each node runs these components to maintain running pods and provide the Kubernetes runtime environment.

### kubelet

The primary node agent. Communicates with the API server. Ensures containers described in PodSpecs are running and healthy.

**Key responsibilities:**
- Watch API server for Pods assigned to its node
- Start/stop containers via the Container Runtime Interface (CRI)
- Run liveness/readiness/startup probes
- Report node and pod status back to the API server
- Mount volumes
- Garbage collect unused images and containers

> **Note:** Does NOT manage containers not created by Kubernetes.

**Port:** `10250` (HTTPS)

```yaml
configFile: /var/lib/kubelet/config.yaml
cgroupDriver: systemd   # must match container runtime (containerd)
containerRuntimeEndpoint: unix:///var/run/containerd/containerd.sock
```

---

### kube-proxy

Runs on each node and maintains network rules for Services. Implements the Service abstraction (ClusterIP / NodePort).

**Modes:**

| Mode | Notes |
|---|---|
| `iptables` | Default; scales to ~5,000 nodes |
| `ipvs` | Better performance; scales to 10,000+ nodes |

> For large clusters or with service meshes (Istio, Cilium), kube-proxy may be replaced by the CNI plugin directly.

---

### Container Runtime

The software that actually runs containers. Kubernetes uses the Container Runtime Interface (CRI) to communicate with it.

**Supported runtimes:**

| Runtime | Notes |
|---|---|
| `containerd` | Default; used by most distros and cloud providers |
| `CRI-O` | OCI-compliant; used by OpenShift |
| `Docker` | No longer supported as CRI in k8s 1.24+ |

> `containerd` uses `runc` (OCI runtime) to actually run containers.

---

## Add-ons

Add-ons extend Kubernetes functionality. They run inside the cluster as Pods/Deployments in the `kube-system` namespace.

### CoreDNS *(mandatory)*

Provides DNS-based service discovery. Every pod gets CoreDNS as its nameserver.

- Default DNS domain: `svc.cluster.local`
- Image: `registry.k8s.io/coredns/coredns:v1.11.1`
- Replicas: `2`

### CNI Plugin *(mandatory)*

Implements the Container Network Interface; gives each pod an IP address and enables pod-to-pod communication.

**Popular CNI plugins:**

| Plugin | Notes |
|---|---|
| Calico | Widely used; supports NetworkPolicy; BGP routing |
| Cilium | eBPF-based; high performance; Hubble observability |
| Flannel | Simple overlay; no NetworkPolicy support |
| Weave | Simple; supports NetworkPolicy |
| AWS VPC CNI | Native VPC IPs for EKS pods |
| Azure CNI | Native VNet IPs for AKS pods |

### metrics-server *(optional, recommended)*

Collects CPU/memory metrics from kubelets. Required for `kubectl top` and HorizontalPodAutoscaler.

- Image: `registry.k8s.io/metrics-server/metrics-server:v0.7.0`

### Kubernetes Dashboard *(optional)*

- Image: `kubernetesui/dashboard:v2.7.0`
- Access via `kubectl proxy`, then open:
  ```
  http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
  ```

### Ingress Controller *(optional)*

Implements the Ingress resource for HTTP/S routing.

**Common choices:**
- NGINX Ingress Controller
- Traefik
- AWS ALB Ingress Controller
- GKE Ingress

### cert-manager *(optional)*

Automates TLS certificate management. Integrates with Let's Encrypt, Vault, and AWS ACM.

- Image: `quay.io/jetstack/cert-manager-controller:v1.14.3`

---

## Cluster Communication Overview

```
kubectl  ──HTTPS──►  kube-apiserver  ──►  etcd (state)
                           │
                 ┌─────────┼─────────┐
                 ▼         ▼         ▼
         kube-scheduler  kube-controller-manager  cloud-controller-manager
                                     │
                           (assigns pods to nodes)
                                     │
                         ┌───────────┴──────────┐
                         ▼                      ▼
                  Worker Node A          Worker Node B
                 ┌──────────────┐       ┌──────────────┐
                 │ kubelet      │       │ kubelet      │
                 │ kube-proxy   │       │ kube-proxy   │
                 │ containerd   │       │ containerd   │
                 │ CNI plugin   │       │ CNI plugin   │
                 │ Pod A1  A2   │       │ Pod B1  B2   │
                 └──────────────┘       └──────────────┘
```

**Pod networking (CNI):**
- Every pod gets a unique cluster-wide IP
- Pods can communicate with any other pod directly (no NAT)
- Services proxy traffic to pods using kube-proxy rules

---

## Useful kubectl Commands

### Cluster Info
```bash
kubectl cluster-info
kubectl cluster-info dump
```

### Nodes
```bash
kubectl get nodes
kubectl get nodes -o wide
kubectl describe node <node-name>
kubectl top nodes                         # requires metrics-server
```

### Control Plane Pods
```bash
kubectl get pods -n kube-system
kubectl get pods -n kube-system -o wide
```

### Component Status
```bash
kubectl get componentstatuses             # deprecated but still useful
```

### Inspect kube-apiserver Flags
```bash
kubectl describe pod kube-apiserver-<node> -n kube-system
```

### Node Maintenance
```bash
# Drain a node (evicts all pods)
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data

# Re-enable scheduling after maintenance
kubectl uncordon <node-name>

# Cordon (stop new pods, keep existing ones running)
kubectl cordon <node-name>
kubectl uncordon <node-name>
```