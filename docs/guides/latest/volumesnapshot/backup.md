## How VolumeSnapshot works in Stash

This guide will show you how Stash takes snapshot of PersistentVolumeClaims using Kubernetes VolumeSnasphot API. 

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
    * [BackupConfiguration]()
    * [BackupSession]()
* You also should be familiar with the following Kubernetes concepts:
    * [VolumeSnapshotContent](https://kubernetes.io/docs/concepts/storage/volume-snapshots/#volume-snapshot-contents)
    * [VolumeSnapshot](https://kubernetes.io/docs/concepts/storage/volume-snapshots/#volumesnapshots)
    * [VolumeSnapshotClass](https://kubernetes.io/docs/concepts/storage/volume-snapshot-classes/)
    You need to create a VolumeSnapshotClass with specifies the `spec.snapshotter` field to point to respective CSI driver.

### Overview

The following diagram shows how Stash creates a VolumeSnapshot via Kubernetes native API. Open the image in a new tab to see the enlarged image.

<p align="center">
  <img alt="Stash Backup Flow" src="/docs/images/v1beta1/backends/volumesnapshot/volume-snapshot-overview.svg">
<figcaption align="center">Fig: VolumeSnapshot works in stash</figcaption>
</p>

The Volume Snapshotting process consists of the following steps:

1. At first, a user creates a `BackupConfiguration` crd which specifies the targeted workload or targeted PVC.

2. Stash operator watches for `BackupConfiguration` crd. 

3. Once it found a `BackupConfiguration` crd, it creates a CronJob that triggers backup on each schedule by creating a `BackupSession` crd. 

4. Stash operator also watches for `BackupSession` crd.

5. Once it found a `BackupSession` crd, it creates a Volume Snapshotter `Job`.

6. Then, Volume Snapshotter Job creates `VolumeSnapshot` crd for each PVC of the target and waits for the CSI driver to complete snapshotting. These `VolumeSnasphot` crd names follow the following format `<PVC name>-<BackupSession creation timestamp in Unix epoch seconds>`.

7. CSI `external-snapshotter` controller watches for `VolumeSnapshot`.

8. Once it found a `VolumeSnapshot`, it triggers `CreateSnapshot` operations against the CSI endpoint.

9. And it also stores data into the Cloud Volume.

10. Once the snapsotting is completed, Stash Operator updates the `status.phase` field of the`BackupSession` crd.

## Next Steps

1. See a step by step guide to snapshot a Deployment's volumes from [here](docs/guides/latest/volumesnapshot/deployment-backup.md).

2. See a step by step guide to snapshot a StatefulSet's volumes from [here]().

3. See a step by step guide to snapshot a stand-alone PVC from [here]().