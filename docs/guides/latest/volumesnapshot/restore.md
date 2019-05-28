## Restore from Volume Snapshot using Stash

This Section will show you how to use Stash for restoring Volume Snapshots via kubernetes native API. 

#### Requirements
At first, you need to have a kubernetes cluster and ensure that a CSI driver that implements snapshots is deployed on your cluster. For testing purpose, we will use [GCE Persistent Disk CSI Driver](https://github.com/kubernetes-sigs/gcp-compute-persistent-disk-csi-driver).The `kubectl` command-line tool must be configured to communicate with your cluster.

* You need to enable the Kubernetes Volume Snapshotting feature via new Kubernetes feature gate 
    * `--feature-gates=VolumeSnapshotDataSource=true`
* Install `Stash` in your cluster following the steps [here](https://appscode.com/products/stash/0.8.3/setup/install/)
* You should be familiar with the following Stash concepts:
    * [RestoreSession]()
    * [VolumeSnapshotContent](https://kubernetes.io/docs/concepts/storage/volume-snapshots/#volume-snapshot-contents)
    * [VolumeSnapshot](https://kubernetes.io/docs/concepts/storage/volume-snapshots/#volumesnapshots)
    * [VolumeSnapshotClass](https://kubernetes.io/docs/concepts/storage/volume-snapshot-classes/)


#### Overview

The following diagram shows how Stash restore from Volume Snapshot via kubernetes native API. Open the image in a new tab to see the enlarged image.

<p align="center">
  <img alt="Stash Backup Flow" src="/docs/images/v1beta1/backends/volumesnapshot/restore-vs-overview.svg">
</p>

The Restore process consists of the following steps:

1. A user creates a `RestoreSession` crd that specifies `volumeClaimTemplates` which holds the information about `VolumeSnapshot` in `dataSource` field.

2. Stash operator watches for `RestoreSession` crd. Once, it found a `RestoreSession` crd, it creates a Restore Job which will create Persistent Volume Claim for restore.

3. CSI `external-snapshotter` controller watches for PVC. Once, it found a PVC, it reads all information about `VolumeSnapshot` and downloads respective data from cloud storage. Afterall CSI `external-snapshotter` resotres data to PVC 
4. Finally Restore job updates the status of the `BackupSession` crd. 
