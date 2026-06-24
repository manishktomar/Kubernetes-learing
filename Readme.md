# Kubernetes Learning Reference

A structured set of annotated YAML files for learning Kubernetes concepts, organized by topic. Each file contains multiple real-world examples with inline comments explaining every important field.

---

## Directory Structure

```
kubernetes/
├── Cluster-Architecture.yaml   Control plane, node components, add-ons
│
├── Installation/               How to get a cluster running
│   ├── Readme.md
│   ├── local-installation.yaml Minikube, kind, k3s, k3d
│   ├── eks.yaml                AWS EKS (eksctl ClusterConfig)
│   ├── aks.yaml                Azure AKS (az CLI + ARM reference)
│   └── gks.yaml                Google GKE (gcloud CLI + Config Connector)
│
├── Workloads/                  Running containerised applications
│   ├── Deployment.yaml         Rolling updates, Recreate, affinity, canary
│   ├── StatefulSet.yaml        Stable identity, ordered scaling, MySQL/ZK
│   ├── DaemonSet.yaml          One pod per node, log collector, GPU plugin
│   ├── CronJob.yaml            Job, parallel Job, indexed Job, CronJob
│   ├── Init-Containers.yaml    Pre-start tasks, dependency waiting
│   ├── Sidecar-Containers.yaml Log shipper, proxy, config sync, native sidecar
│   ├── Namespace.yaml          Isolation, ResourceQuota, LimitRange, NetworkPolicy
│   └── Others.yaml             Pod, ReplicaSet, SecurityContext, HPA, PDB, PriorityClass
│
├── Configuration/              Externalizing app config and health
│   ├── Readme.md
│   ├── ConfigMaps.yaml         Non-sensitive config, env vars, volume mounts
│   ├── Secrets.yaml            Sensitive data, TLS, Docker registry, SSH
│   ├── Liveness,-Readiness-Startup-Probes.yaml  HTTP, exec, TCP, gRPC probes
│   └── Resource-Management-for-Pods-Containers.yaml  requests/limits, QoS, LimitRange
│
├── Storage/                    Persistent and ephemeral storage
│   ├── Readme.md
│   ├── Volumes.yaml            emptyDir, hostPath, configMap, secret, downwardAPI
│   ├── Persistent-Volumes.yaml PV definitions (hostPath, NFS, EBS, Azure, GCE)
│   ├── PVC.yaml                PVC requests, snapshots, volume expansion
│   └── Storage-Classes.yaml   Dynamic provisioning (gp3, Azure, GKE, NFS, local)
│
└── Networking/                 Services and cluster networking
    ├── Readme.md
    ├── svc-ClusterIP.yaml      Internal service, headless, manual Endpoints
    ├── svc-NodePort.yaml       Expose on node ports 30000-32767
    ├── svc-LoadBalancer.yaml   Cloud LB (AWS NLB, Azure, GKE, MetalLB)
    └── svc-ExternalName.yaml   DNS alias / CNAME for external services
```

---

## Learning Path

Start here if you're new to Kubernetes:

1. **Cluster Architecture** — understand what components exist and why
2. **Installation** — get a local cluster (minikube or kind)
3. **Workloads / Deployment** — deploy your first app
4. **Networking / ClusterIP + NodePort** — expose it
5. **Configuration / ConfigMaps + Secrets** — externalize config
6. **Storage / Volumes + PVC** — add persistence
7. **Workloads / StatefulSet** — run stateful apps (databases)
8. **Configuration / Probes** — make health checks work
9. **Configuration / Resources** — set requests and limits
10. **Workloads / CronJob** — schedule batch work
11. **Workloads / DaemonSet** — run cluster-wide agents
12. **Networking / LoadBalancer** — expose to the internet
13. **Workloads / Others** — HPA, PDB, security contexts

---

## Common kubectl Commands

```bash
# Cluster
kubectl cluster-info
kubectl get nodes -o wide
kubectl top nodes                   # requires metrics-server

# Apply / delete manifests
kubectl apply -f <file>.yaml
kubectl apply -f <directory>/
kubectl delete -f <file>.yaml

# Inspect resources
kubectl get all -n default
kubectl describe pod <pod-name>
kubectl logs <pod-name> [-c <container>] [-f]
kubectl exec -it <pod-name> -- bash

# Deployments
kubectl rollout status deployment/<name>
kubectl rollout history deployment/<name>
kubectl rollout undo deployment/<name>

# Scale
kubectl scale deployment <name> --replicas=5

# Port forwarding (local development)
kubectl port-forward svc/<service> 8080:80
kubectl port-forward pod/<pod> 8080:8080

# Namespace
kubectl get all -n <namespace>
kubectl config set-context --current --namespace=<ns>

# Secrets / ConfigMaps
kubectl get secrets
kubectl get configmaps
kubectl get secret <name> -o jsonpath='{.data.<key>}' | base64 -d

# Storage
kubectl get pv,pvc
kubectl get storageclass

# Troubleshooting
kubectl describe pod <pod>          # events section is key
kubectl get events --sort-by=.lastTimestamp
kubectl top pods --containers
```

---

## Key Concepts Cheat Sheet

| Concept | What it does |
|---|---|
| Pod | Smallest deployable unit; one or more containers |
| Deployment | Manages ReplicaSets; rolling updates; rollback |
| StatefulSet | Pods with stable identity and persistent storage |
| DaemonSet | One pod per node |
| Job / CronJob | Run to completion; scheduled batch |
| Service | Stable network endpoint + load balancing for pods |
| ConfigMap | Externalise non-sensitive config |
| Secret | Store sensitive data (base64-encoded) |
| PersistentVolume | Cluster-level storage resource |
| PersistentVolumeClaim | Request storage from a PV |
| StorageClass | Recipe for dynamic PV provisioning |
| Namespace | Logical isolation boundary inside a cluster |
| RBAC | Role-Based Access Control (who can do what) |
| NetworkPolicy | Firewall rules for pod traffic |
| HPA | Auto-scale pods based on metrics |
| Ingress | HTTP/S routing rules (host + path → service) |

---

## Useful Resources

- [Kubernetes Official Docs](https://kubernetes.io/docs/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Kubernetes API Reference](https://kubernetes.io/docs/reference/kubernetes-api/)
- [Play with Kubernetes](https://labs.play-with-k8s.com/)
- [Killercoda K8s Scenarios](https://killercoda.com/kubernetes)
