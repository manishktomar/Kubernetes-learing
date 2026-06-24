# Kubernetes Configuration

This directory covers how to configure Kubernetes workloads — externalizing application config, storing secrets securely, and managing container health and resources.

---

## Files

| File | Description |
|---|---|
| [ConfigMaps.yaml](ConfigMaps.yaml) | Storing non-sensitive config data; env vars, volume mounts, immutable CMs |
| [Secrets.yaml](Secrets.yaml) | Storing sensitive data; Opaque, TLS, Docker registry, SSH secrets |
| [Liveness,-Readiness-Startup-Probes.yaml](Liveness,-Readiness-Startup-Probes.yaml) | HTTP, exec, TCP socket, gRPC health probes |
| [Resource-Management-for-Pods-Containers.yaml](Resource-Management-for-Pods-Containers.yaml) | CPU/memory requests & limits, QoS classes, LimitRange, ResourceQuota |

---

## ConfigMaps

ConfigMaps decouple configuration from your container image. They store **non-sensitive** key-value pairs or config files.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: "production"
  APP_PORT: "8080"
  config.yaml: |
    key: value
```

**Consumption methods:**
- Single env var: `valueFrom.configMapKeyRef`
- All keys as env vars: `envFrom.configMapRef`
- Volume mount: `volumes[].configMap`

---

## Secrets

Secrets store **sensitive** data. Values are **base64-encoded** (not encrypted by default).

```bash
# Create from literals
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=secret123

# Decode a value
kubectl get secret db-secret -o jsonpath='{.data.password}' | base64 -d
```

> **Production:** Use [External Secrets Operator](https://external-secrets.io) to sync from AWS Secrets Manager / Azure Key Vault / HashiCorp Vault. Enable **Encryption at Rest** in your cluster's `EncryptionConfiguration`.

**Secret types:**

| Type | Purpose |
|---|---|
| `Opaque` | General-purpose (default) |
| `kubernetes.io/tls` | TLS certificate + key |
| `kubernetes.io/dockerconfigjson` | Private image registry credentials |
| `kubernetes.io/basic-auth` | Username + password |
| `kubernetes.io/ssh-auth` | SSH private key |

---

## Health Probes

| Probe | Failure action | Use for |
|---|---|---|
| **Liveness** | Restart container | Detecting deadlocks / hung processes |
| **Readiness** | Remove from Service endpoints | Temporary unavailability (loading cache, DB migrations) |
| **Startup** | Delay liveness/readiness | Slow-starting apps (JVM, large ML models) |

**Probe mechanisms:**

```yaml
# HTTP GET
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 15
  failureThreshold: 3

# Exec command
readinessProbe:
  exec:
    command: ["redis-cli", "ping"]

# TCP socket
livenessProbe:
  tcpSocket:
    port: 6379

# gRPC (k8s 1.27+)
livenessProbe:
  grpc:
    port: 50051
```

---

## Resource Management

Every container should define `requests` and `limits`.

```yaml
resources:
  requests:        # guaranteed — used for scheduling
    cpu: "100m"    # 0.1 CPU core
    memory: "128Mi"
  limits:          # maximum — enforced at runtime
    cpu: "250m"
    memory: "256Mi"
```

**QoS Classes (determined automatically):**

| Class | Condition | Eviction priority |
|---|---|---|
| `Guaranteed` | requests == limits for all resources | Last evicted |
| `Burstable` | requests < limits (or partial) | Middle |
| `BestEffort` | No requests or limits | First evicted |

**Check resource usage:**
```bash
kubectl top nodes
kubectl top pods --containers
kubectl describe pod <pod-name>   # shows QoS Class
```

---

## Quick Reference

```bash
# ConfigMaps
kubectl create configmap my-cm --from-literal=key=value
kubectl get configmap my-cm -o yaml

# Secrets
kubectl create secret generic my-secret --from-literal=pass=secret
kubectl get secret my-secret -o jsonpath='{.data.pass}' | base64 -d

# Probes – check failure events
kubectl describe pod <pod-name>   # look for "Liveness probe failed" in Events

# Resources – check QoS
kubectl describe pod <pod-name>   # look for "QoS Class:"
kubectl top pods
```
