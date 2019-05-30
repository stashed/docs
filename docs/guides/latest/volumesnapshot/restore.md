## Restore PVC from VolumeSnapshot using Stash

This guide will show you how Stash restore PersistentVolumeClaims from snapshot using Kubernetes VolumeSnapshot API. 

### Requirements

* At first, you need to have a Kubernetes cluster and ensure that a CSI driver that implements snapshots is deployed on your cluster. The following CSI drivers support snapshots:
    * [GCE Persistent Disk CSI Driver](https://github.com/kubernetes-sigs/gcp-compute-persistent-disk-csi-driver)
    * [OpenSDS CSI Driver](https://github.com/opensds/nbp/tree/master/csi/server)
    * [Ceph RBD CSI Driver](https://github.com/ceph/ceph-csi/tree/master/pkg/rbd)
    * [Portworx CSI Driver](https://github.com/libopenstorage/openstorage/tree/master/csi)
For testing purposes, we will use GCE Persistent Disk CSI Driver. 
* The `kubectl` command-line tool must be configured to communicate with your cluster.

* You need to enable the Kubernetes VolumeSnapshotDataSource alpha feature via new Kubernetes feature gate 
    * `--feature-gates=VolumeSnapshotDataSource=true`
* Install `Stash` in your cluster following the steps [here](https://appscode.com/products/stash/0.8.3/setup/install/)
* You should be familiar with the following Stash concepts:
    * [RestoreSession]()
* You also should be familiar with the following Kubernetes concepts:
    * [VolumeSnapshotContent](https://kubernetes.io/docs/concepts/storage/volume-snapshots/#volume-snapshot-contents)
    * [VolumeSnapshot](https://kubernetes.io/docs/concepts/storage/volume-snapshots/#volumesnapshots)
    * [VolumeSnapshotClass](https://kubernetes.io/docs/concepts/storage/volume-snapshot-classes/)

### Overview

The following diagram shows how Stash restore PVC from snapshot using Kubernetes VolumeSnapshot API. Open the image in a new tab to see the enlarged image.

<p align="center">
  <img alt="Stash Backup Flow" src="/docs/images/v1beta1/backends/volumesnapshot/restore-vs-overview.svg">
</p>

The Restoring process consists of the following steps:

1. At first, a user creates a `RestoreSession` crd which specifies the `volumeClaimTemplates`. `volumeClaimTemplates` holds the information about `VolumeSnapshot`.

2. Stash operator watches for `RestoreSession` crd. 

3. Once it found a `RestoreSession` crd, it creates a Restore Job.

4. Restore Job creates PersistentVolumeClaim for restore.

5. CSI `external-snapshotter` controller watches for PVC. 

6. Once it found a new PVC, it reads all information from `VolumeSnapshot` 

7. And downloads respective data from cloud storage.

8. Afterall CSI `external-snapshotter` restores PVC from VolumeSnapshot.

4. Once restoring process is completed, the Stash operator updates the `status.phase` field of the `BackupSession` crd. 

## Next Steps

1. See a step by step guide to snapshot a Deployment's volumes from [here](docs/guides/latest/volumesnapshot/deployment-backup.md).

2. See a step by step guide to snapshot a StatefulSet's volumes from [here]().

3. See a step by step guide to snapshot a stand-alone PVC from [here]().
