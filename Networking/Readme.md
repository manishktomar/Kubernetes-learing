# Kubernetes Networking

This directory covers Kubernetes networking — how traffic flows between pods, within the cluster, and from the outside world.

---

## Files

| File | Description |
|---|---|
| [svc-ClusterIP.yaml](svc-ClusterIP.yaml) | Internal-only service; headless service; manual Endpoints / ExternalName |
| [svc-NodePort.yaml](svc-NodePort.yaml) | Expose service on a port on every node |
| [svc-LoadBalancer.yaml](svc-LoadBalancer.yaml) | Cloud load balancer; AWS NLB, Azure internal LB, GKE BackendConfig, MetalLB |
| [svc-ExternalName.yaml](svc-ExternalName.yaml) | DNS alias (CNAME) for external services or cross-namespace routing |

---

## Service Types

| Type | Accessibility | Use case |
|---|---|---|
| **ClusterIP** | Cluster only | Internal microservice communication |
| **NodePort** | Cluster + Node IPs | Dev/test; on-prem without a cloud LB |
| **LoadBalancer** | Internet (external IP) | Production cloud-hosted services |
| **ExternalName** | Cluster only (CNAME) | Abstract external services; cross-ns routing |

---

## Service Traffic Flow

```
LoadBalancer
  │  Internet traffic
  ▼
NodePort  (<NodeIP>:30000-32767)
  │  kube-proxy routing
  ▼
ClusterIP  (virtual IP inside cluster)
  │  iptables / IPVS rules
  ▼
Pod  (matched by selector labels)
```

---

## ClusterIP — Internal Services

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  type: ClusterIP        # default
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 8080
```

DNS name: `backend.default.svc.cluster.local` (or just `backend` within same namespace)

**Headless** (`clusterIP: None`) — no virtual IP; DNS returns pod IPs directly. Required for StatefulSets.

---

## NodePort — Dev / On-Prem

```yaml
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080     # omit for auto-assign (30000-32767)
```

```bash
curl http://<any-node-ip>:30080
minikube service my-svc --url   # get URL on minikube
```

---

## LoadBalancer — Production Cloud

```yaml
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 8080
```

```bash
kubectl get svc web-lb -w    # wait for EXTERNAL-IP
curl http://<external-ip>/
```

Cloud-specific annotations:
- **AWS**: `service.beta.kubernetes.io/aws-load-balancer-type: "external"` (NLB)
- **Azure**: `service.beta.kubernetes.io/azure-load-balancer-internal: "true"` (internal LB)
- **GKE**: `cloud.google.com/backend-config` (BackendConfig)

---

## ExternalName — DNS Alias

```yaml
spec:
  type: ExternalName
  externalName: prod-db.example.com
```

Pods connect to `my-svc.default.svc.cluster.local` → kube-dns returns CNAME `prod-db.example.com`.

---

## Key Networking Concepts

### 1. Services & Endpoints
Every Service has an associated **Endpoints** (or EndpointSlice) object listing the pod IPs currently matched by the selector.

```bash
kubectl get endpoints my-svc
kubectl describe svc my-svc    # shows Endpoints line
```

### 2. kube-proxy
Runs on every node. Programs iptables (or IPVS) rules to handle ClusterIP/NodePort routing. Traffic never actually passes through kube-proxy — it's handled in-kernel.

### 3. DNS (CoreDNS)
Every pod gets the cluster's DNS server injected via `/etc/resolv.conf`. DNS search path includes the pod's own namespace.

```
# Full DNS name
<svc>.<namespace>.svc.cluster.local

# Short forms (within same namespace)
<svc>
<svc>.<namespace>
```

### 4. Network Policies
Restrict traffic between pods. Requires a CNI plugin that enforces them (Calico, Cilium, Weave).

```yaml
kind: NetworkPolicy
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
```

### 5. Ingress (HTTP/S routing)
A single LoadBalancer IP can serve many services via host/path-based routing. Requires an Ingress Controller (NGINX, Traefik, ALB, etc.).

```yaml
kind: Ingress
spec:
  rules:
    - host: api.example.com
      http:
        paths:
          - path: /
            backend:
              service:
                name: api-svc
                port:
                  number: 80
```

---

## Quick Reference

```bash
# Services
kubectl get svc
kubectl describe svc my-svc
kubectl get endpoints my-svc

# Test internal DNS resolution
kubectl run test --image=busybox:1.36 --restart=Never -it --rm \
  -- nslookup backend.default.svc.cluster.local

# Port-forward (any service, dev use)
kubectl port-forward svc/backend 8080:80

# NodePort access
curl http://$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[0].address}'):30080

# LoadBalancer external IP
kubectl get svc web-lb -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
```
