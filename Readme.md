Here is the complete, raw markdown code for your `README.md`. You can copy everything inside the block below and paste it directly into your file:

```markdown
# ☸️ Kubernetes Learning Reference

A structured set of annotated YAML files for learning Kubernetes concepts, organized by topic. Each file contains multiple real-world examples with inline comments explaining every important field.

---

## 🗺️ Learning Path

Follow this structured roadmap to master the Kubernetes ecosystem systematically:

1. **Cluster Architecture** — Understand what components exist and why.
2. **Installation** — Get a local cluster running (Minikube, Kind, K3s, or K3d).
3. **Workloads / Deployment** — Deploy your very first containerized application.
4. **Networking / ClusterIP + NodePort** — Route internal traffic and expose your app.
5. **Configuration / ConfigMaps + Secrets** — Externalize configuration and sensitive variables.
6. **Storage / Volumes + PVC** — Introduce persistent storage states.
7. **Workloads / StatefulSet** — Spin up stateful applications like databases (MySQL, ZooKeeper).
8. **Configuration / Probes** — Implement health checks (Liveness, Readiness, Startup).
9. **Configuration / Resources** — Define hard compute constraints (Requests and Limits).
10. **Workloads / CronJob** — Schedule automated batch execution processes.
11. **Workloads / DaemonSet** — Run universal, cluster-wide background agents.
12. **Networking / LoadBalancer** — Expose your services natively to the public internet.
13. **Workloads / Others** — Scale and protect workloads using HPA, PDB, and SecurityContexts.

---

## 📂 Kubernetes Repository Structure

Use the interactive directory mapping below to navigate directly to specific configuration templates.

* [🏛️ Cluster Architecture](#-cluster-architecture) — *Control plane, node components, add-ons*
* [🚀 Installation & Bootstrapping](#-installation--bootstrapping) — *How to get a cluster running*
* [📦 Workloads & Orchestration](#-workloads--orchestration) — *Running containerized applications*
* [⚙️ Configuration & Security](#%EF%B8%8F-configuration--security) — *Externalizing app config and health*
* [💾 Storage Subsystems](#-storage-subsystems) — *Persistent and ephemeral storage*
* [🌐 Cluster Networking](#-cluster-networking) — *Services and cluster networking*

---

### 🏛️ Cluster Architecture
* **File:** `Cluster-Architecture.yaml`
* **Focus:** Deep dive into Control Plane internals, worker node operations, and baseline cluster add-ons.

### 🚀 Installation & Bootstrapping
> Setup and bootstrap configurations for various cloud providers and local sandboxes.
* 📋 `Readme.md` — Environment setup instructions.
* 💻 `local-installation.yaml` — Local cluster scripts for **Minikube**, **Kind**, **K3s**, and **K3d**.
* ☁️ `eks.yaml` — AWS EKS infrastructure using declarative `eksctl` ClusterConfigs.
* ☁️ `aks.yaml` — Azure AKS configuration via `az CLI` instructions and ARM syntax.
* ☁️ `gks.yaml` — Google GKE deployment setups via `gcloud CLI` and Config Connector specs.

### 📦 Workloads & Orchestration
> Resource configurations for scheduling, executing, and scaling container state models.
* 📄 `Deployment.yaml` — Rolling updates, Recreate strategies, node affinity configurations, and canary rollouts.
* 📄 `StatefulSet.yaml` — Stable network identifiers, ordinal scaling arrays, and persistent database clustering.
* 📄 `DaemonSet.yaml` — Node-locked singleton instances (e.g., fluentd log collectors, GPU hardware access plugins).
* 📄 `CronJob.yaml` — Run-to-completion Jobs, parallel/indexed batch executions, and cron interval engines.
* 📄 `Init-Containers.yaml` — Blocking pre-start dependency validation routines.
* 📄 `Sidecar-Containers.yaml` — Operational sidecars, log shippers, proxies, and Kubernetes native sidecar lifecycles.
* 📄 `Namespace.yaml` — Hard isolation, resource constraints (`ResourceQuota`), range limits (`LimitRange`), and `NetworkPolicies`.
* 📄 `Others.yaml` — Primitive items: Pods, ReplicaSets, `SecurityContext` parameters, `HPA`, `PDB`, and `PriorityClass`.

### ⚙️ Configuration & Security
> Decoupling application business logic from structural environments and secrets.
* 📋 `Readme.md` — Configuration architecture workflows.
* 📄 `ConfigMaps.yaml` — Plaintext properties maps, application settings files, and volume environment attachments.
* 📄 `Secrets.yaml` — Base64 hidden keys, TLS keys, private Docker Registry auth blocks, and SSH configurations.
* 📄 `Liveness,-Readiness-Startup-Probes.yaml` — Application health evaluation using HTTP, Exec, TCP, and gRPC endpoints.
* 📄 `Resource-Management-for-Pods-Containers.yaml` — Microcompute controls via CPU/Memory requests/limits and QoS policies.

### 💾 Storage Subsystems
> Declaring persistent storage states and dynamic runtime attachments.
* 📋 `Readme.md` — Decoupling persistent storage layers.
* 📄 `Volumes.yaml` — Ephemeral internal storage mounts (`emptyDir`, `hostPath`, `configMap`, `secret`, `downwardAPI`).
* 📄 `Persistent-Volumes.yaml` — Infrastructure storage objects (NFS, EBS, Azure Disk, GCE Persistent Disk).
* 📄 `PVC.yaml` — Consumption request profiles, snapshot configurations, and volume extension setups.
* 📄 `Storage-Classes.yaml` — Automated structural storage providers (`gp3`, Azure managed, GKE storage systems).

### 🌐 Cluster Networking
> Exposing ephemeral containers through reliable, load-balanced connectivity wrappers.
* 📋 `Readme.md` — Pod-to-Pod and External-to-Internal networking topologies.
* 📄 `svc-ClusterIP.yaml` — Internal load balancing, headless routing entries, and hardcoded manual endpoints.
* 📄 `svc-NodePort.yaml` — Broad external port routing bindings directly on worker nodes (`30000-32767`).
* 📄 `svc-LoadBalancer.yaml` — Auto-provisioned Layer-4 cloud load balancers (AWS NLB, Azure Load Balancer, MetalLB).
* 📄 `svc-ExternalName.yaml` — Pure application-level CNAME aliases redirecting to outside domains (e.g., cloud databases).

---

## 🛠️ Essential kubectl Cheat Sheet

### 🔍 Discovery & Infrastructure Checks
```bash
kubectl cluster-info                           # View active control plane location details
kubectl get nodes -o wide                      # Fetch detailed state parameters of worker nodes
kubectl top nodes                              # Monitor host resource strain (Requires Metrics-Server)

```

### 🏎️ Declarative Manifest Management

```bash
kubectl apply -f <filename>.yaml               # Create or modify resources declaratively
kubectl apply -f <directory>/                  # Recursively parse and apply an entire directory
kubectl delete -f <filename>.yaml              # Safely tear down resources

```

### 🔬 Inspection & Application Debugging

```bash
kubectl get all -n default                     # Snapshot default running elements
kubectl describe pod <pod-name>                # Deep-dive inspect a specific pod (Review "Events" for errors)
kubectl logs <pod-name> [-c <container>] [-f]  # Stream container engine standard output logs
kubectl exec -it <pod-name> -- bash            # Attach a local interactive shell to a running container

```

### 🔄 Deployment Orchestration

```bash
kubectl rollout status deployment/<name>       # Follow rolling update migration steps live
kubectl rollout history deployment/<name>      # Inspect the revision timeline tracking
kubectl rollout undo deployment/<name>         # Rollback immediately to the previous deployment layer
kubectl scale deployment <name> --replicas=5   # Re-dimension stateless pool footprints

```

### 🔌 Port Forwarding (Local Development)

```bash
kubectl port-forward svc/<service-name> 8080:80 # Map local ports to a specific Service definition
kubectl port-forward pod/<pod-name> 8080:8080  # Direct bypass link into an isolated Pod

```

### 🔑 Security & Configuration Utilities

```bash
kubectl get secrets
kubectl get configmaps
# Extract and decode base64 fields natively out of secret files
kubectl get secret <name> -o jsonpath='{.data.<key>}' | base64 -d

```

### 💾 Storage Auditing

```bash
kubectl get pv,pvc                             # Cross-reference resource allocations against actual requests
kubectl get storageclass                       # Audit registered dynamic cloud storage factories

```

---

## 🧠 Key Concepts Cheat Sheet

| Component | Operational Responsibility | Lifecycle State |
| --- | --- | --- |
| **Pod** | Smallest single deployment wrapper; schedules tight container collections | Volatile |
| **Deployment** | Handles ReplicaSets; orchestrates automated, zero-downtime application updates | Stateless |
| **StatefulSet** | Grants stable persistent storage bindings and persistent network identity markers | Stateful |
| **DaemonSet** | Distributes specific singular operational instances onto every matching worker node | Infrastructure |
| **Job / CronJob** | Drives transient, scheduled run-to-completion execution tracks | Short-Lived |
| **Service** | Establishes fixed internal IP addressing schemes and load balancing configurations | Network Target |
| **ConfigMap** | Keeps infrastructure settings detached from compiled codebase configurations | Configuration |
| **Secret** | Isolates critical credential values securely using basic Base64 protections | Configuration |
| **PersistentVolume** | Low-level storage allocations mapped straight onto real computing infrastructure | Storage Plane |
| **PersistentVolumeClaim** | Individual application resource authorization requests for persistent disk layers | Storage Plane |
| **StorageClass** | Dynamic factory parameters that auto-provision external backing disks on demand | Storage Plane |
| **Namespace** | Creates logical sandboxes and tenant dividers inside a shared physical environment | Boundary |

---

## 🔗 Useful Learning Resources

* 📖 [Kubernetes Official Docs](https://kubernetes.io/docs/) — *The absolute reference documentation guide.*
* 🏎️ [kubectl Cheat Sheet Reference](https://kubernetes.io/docs/reference/kubectl/cheatsheet/) — *Quick command syntaxes.*
* 🧪 [Killercoda K8s Scenarios](https://killercoda.com/kubernetes) — *Interactive, scriptable sandbox scenarios.*
* 🛝 [Play with Kubernetes](https://labs.play-with-k8s.com/) — *A temporary multi-node workspace setup directly inside the browser.*

```

***

What specific Kubernetes topic or concept are you planning to work on first within this repository?

```
