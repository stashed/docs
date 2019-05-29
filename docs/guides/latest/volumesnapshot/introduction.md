# Volume Snapshots

## Introduction

Kubernetes v1.12 introduces alpha support for volume snapshotting. This feature allows creating/deleting volume snapshots, and the ability to create new volumes from a snapshot natively using the Kubernetes API.

Volume Snapshots are only supported by CSI drivers. Kubernetes CSI currently enables CSI drivers to expose the following functionality via the Kubernetes API:

- Creation and deletion of volume snapshots via [Kubernetes native API](https://kubernetes.io/docs/concepts/storage/volume-snapshots/).
- provisioning a new volume pre-populated with the snapshot data or to   restore the existing volume to a previous state represented by the snapshot

#### CSI Driver

To implement the snapshot feature, a CSI driver MUST add support for additional controller capabilities `CREATE_DELETE_SNAPSHOT` and `LIST_SNAPSHOTS`, and implement additional controller RPCs: `CreateSnapshot`, `DeleteSnapshot`, and `ListSnapshots`. For details, see [the CSI spec](https://github.com/container-storage-interface/spec/blob/master/spec.md).

## Volume Snapshots API

#### API

Similar to the API for managing Kubernetes Persistent Volumes, Kubernetes Volume Snapshots introduce three new API objects for managing snapshots:
1. `VolumeSnapshotContent`, 
2. `VolumeSnapshot`, and 
3. `VolumeSnapshotClass` 
 
 are CRDs, not part of the core API. See [ Kubernetes Snapshot documentation](https://kubernetes.io/blog/2018/10/09/introducing-volume-snapshot-alpha-for-kubernetes/) for more details. 

#### SideCar

The CRDs are automatically deployed by the CSI `external-snapshotter` sidecar. The Kubernetes CSI development team maintains the external-snapshotter Kubernetes CSI Sidecar Containers. This sidecar container implements the logic for watching the Kubernetes API for snapshot objects and issuing the appropriate CSI snapshot calls against a CSI endpoint. For more details, see [external-snapshotter documentation](https://github.com/kubernetes-csi/external-snapshotter).

#### Cluster Setup for Volume Snapshot

Before using Kubernetes volume snapshotting, you must:

* Ensure a CSI driver implementing snapshots is deployed and running on your Kubernetes cluster.
* Enable the Kubernetes Volume Snapshotting feature via new Kubernetes feature gate (disabled by default for alpha):
    * Set the following flag on the API server binary:  `--feature-gates=VolumeSnapshotDataSource=true`

