
# Kubernetes Storage

This directory covers all storage primitives in Kubernetes — from ephemeral pod-local storage to durable, cluster-wide persistent volumes.

---

## Files

| File | Description |
|---|---|
| [Volumes.yaml](Volumes.yaml) | emptyDir, hostPath, configMap, secret, projected, downwardAPI, NFS volumes |
| [Persistent-Volumes.yaml](Persistent-Volumes.yaml) | PV definitions — hostPath, NFS, EBS, Azure Disk, GCE PD, Local, Block |
| [PVC.yaml](PVC.yaml) | PVC requests, pod/deployment usage, VolumeSnapshot, volume expansion |
| [Storage-Classes.yaml](Storage-Classes.yaml) | Dynamic provisioning — standard, gp3, Azure, GKE, NFS, local StorageClasses |

---

## Storage Hierarchy

```
StorageClass  ─► dynamic provisioning
     │
     ▼
PersistentVolume (PV)  ─► cluster-level storage resource
     │ bound
     ▼
PersistentVolumeClaim (PVC)  ─► namespace-level storage request
     │ mounted by
     ▼
Pod Volume  ─► per-pod ephemeral or persistent storage
```

---

## Volume Types Summary

| Type | Lifecycle | Multi-pod | Use case |
|---|---|---|---|
| `emptyDir` | Pod lifetime | Same pod only | Scratch space, sidecar file sharing |
| `hostPath` | Node lifetime | Same node only | DaemonSets, node-level access |
| `configMap` | Pod lifetime | Yes | Config files from ConfigMap |
| `secret` | Pod lifetime | Yes | Credentials files from Secret |
| `nfs` | Independent | Yes (RWX) | Shared storage across pods |
| `PVC` | Independent | Depends on access mode | Databases, stateful apps |

---

## Access Modes

| Mode | Abbreviation | Multiple nodes? | Multiple pods? |
|---|---|---|---|
| ReadWriteOnce | RWO | No | Only on one node |
| ReadOnlyMany | ROX | Yes | Yes (read-only) |
| ReadWriteMany | RWX | Yes | Yes |
| ReadWriteOncePod | RWOP | No | Exactly one pod (k8s 1.22+) |

> **Note:** EBS, Azure Disk, GCE PD only support **RWO**. NFS, Azure Files, GCS Filestore support **RWX**.

---

## Reclaim Policies

| Policy | What happens when PVC is deleted |
|---|---|
| `Delete` | PV and underlying disk are deleted automatically |
| `Retain` | PV stays; must be manually cleaned up. Protects data. |
| `Recycle` | Deprecated. Scrubs data and makes PV available again. |

> Use **Retain** for databases and critical data. Use **Delete** for temporary/ephemeral storage.

---

## Dynamic Provisioning (recommended)

```yaml
# 1. Create a StorageClass
kind: StorageClass
provisioner: ebs.csi.aws.com
parameters:
  type: gp3

# 2. Create a PVC referencing the StorageClass
kind: PersistentVolumeClaim
spec:
  storageClassName: gp3
  resources:
    requests:
      storage: 20Gi

# 3. Use in a Pod
volumes:
  - name: data
    persistentVolumeClaim:
      claimName: my-pvc
```
The PV is created automatically — no manual provisioning needed.

---

## volumeBindingMode

| Mode | When PV is provisioned | Recommended for |
|---|---|---|
| `Immediate` | When PVC is created | NFS, local dev |
| `WaitForFirstConsumer` | When pod is scheduled | Cloud block storage (EBS, Azure Disk, GCE PD) |

> Always use **WaitForFirstConsumer** for cloud storage to ensure the volume lands in the same availability zone as the pod.

---

## Quick Reference

```bash
# Storage classes
kubectl get storageclass
kubectl describe sc gp3

# Persistent Volumes
kubectl get pv
kubectl describe pv pv-hostpath
# PV status:  Available | Bound | Released | Failed

# PVCs
kubectl get pvc
kubectl describe pvc basic-pvc
# PVC status: Pending | Bound | Lost

# See PV ↔ PVC binding
kubectl get pv,pvc

# Resize a PVC (StorageClass must have allowVolumeExpansion: true)
kubectl edit pvc dynamic-pvc   # increase storage request

# Force-delete stuck PVC
kubectl patch pvc <name> -p '{"metadata":{"finalizers":null}}'
```
