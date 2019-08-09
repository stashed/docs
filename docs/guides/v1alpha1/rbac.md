---
title: RBAC | Stash
description: RBAC
menu:
  product_stash_v0.9.0-rc.0:
    identifier: rbac-stash
    name: RBAC
    parent: v1alpha1-guides
    weight: 45
product_name: stash
menu_name: product_stash_v0.9.0-rc.0
section_menu_id: guides
---
> New to Stash? Please start [here](/docs/concepts/README.md).

# Configuring RBAC

To use Stash in a RBAC enabled cluster, [install Stash](/docs/setup/install.md) with RBAC options. This creates a ClusterRole named `stash-sidecar`.

Sidecar container added to workloads makes various calls to Kubernetes api. ServiceAccounts used with Deployment, ReplicaSet, DaemonSet and ReplicationController workloads are automatically bound to `stash-sidecar` ClusterRole by Stash operator. Users should manually add the following RoleBinding to service accounts used with StatefulSet workloads to authorize these api calls.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: <statefulset-name>-stash-sidecar
  namespace: <statefulset-namespace>
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: stash-sidecar
subjects:
- kind: ServiceAccount
  name: <statefulset-sa>
  namespace: <statefulset-namespace>
```

You can find full working examples [here](/docs/guides/v1alpha1/workloads.md).

## Next Steps

- Learn how to use Stash to backup a Kubernetes deployment [here](/docs/guides/v1alpha1/backup.md).
- Learn about the details of Restic CRD [here](/docs/concepts/crds/v1alpha1/restic.md).
- To restore a backup see [here](/docs/guides/v1alpha1/restore.md).
- Learn about the details of Recovery CRD [here](/docs/concepts/crds/v1alpha1/recovery.md).
- To run backup in offline mode see [here](/docs/guides/v1alpha1/offline_backup.md)
- See the list of supported backends and how to configure them [here](/docs/guides/v1alpha1/backends/overview.md).
- See working examples for supported workload types [here](/docs/guides/v1alpha1/workloads.md).
- Thinking about monitoring your backup operations? Stash works [out-of-the-box with Prometheus](/docs/guides/v1alpha1/monitoring/overview.md).
- Want to hack on Stash? Check our [contribution guidelines](/docs/CONTRIBUTING.md).
