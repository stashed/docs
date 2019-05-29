## Create Volume Snapshot using Stash

This tutorial will show you how to use Stash for creating Volume Snapshots via kubernetes native API. 

#### Requirements
At first, you need to have a kubernetes cluster and ensure that a CSI driver that implements snapshots is deployed on your cluster. For testing purpose, we will use [GCE Persistent Disk CSI Driver](https://github.com/kubernetes-sigs/gcp-compute-persistent-disk-csi-driver).The `kubectl` command-line tool must be configured to communicate with your cluster.

* You need to enable the Kubernetes Volume Snapshotting feature via new Kubernetes feature gate 
    * `--feature-gates=VolumeSnapshotDataSource=true`
* Install `Stash` in your cluster following the steps [here](https://appscode.com/products/stash/0.8.3/setup/install/)
* You should be familiar with the following Stash concepts:
    * [BackupConfiguration]()
    * [BackupSession]()
    * [VolumeSnapshotContent](https://kubernetes.io/docs/concepts/storage/volume-snapshots/#volume-snapshot-contents)
    * [VolumeSnapshot](https://kubernetes.io/docs/concepts/storage/volume-snapshots/#volumesnapshots)
    * [VolumeSnapshotClass](https://kubernetes.io/docs/concepts/storage/volume-snapshot-classes/)

#### Overview

The following diagram shows how Stash creates a Volume Snapshot via kubernetes native API. Open the image in a new tab to see the enlarged image.

<p align="center">
  <img alt="Stash Backup Flow" src="/docs/images/v1beta1/backends/volumesnapshot/volume-snapshot-overview.svg">
</p>

The Volume Snapshot process consists of the following steps:

1. At first, a user creates a `VolumeSnapshotClass` with specifies the `snapshotter` field to point to determined CSI driver, for GCE Persistent Disk CSI Driver, `snapshotter` will be `pd.csi.storage.gke.io` and also specifies the `annotations` with `snapshot.storage.kubernetes.io/is-default-class: "true"` for default `VolumeSnapshotClass`.

2. Then, the user creates a `BackupConfiguration` crd which specifies the targeted workload PVC or standalone PVC. It also specifies the `VolumeSnapshotClass` information where the snapshots will be stored.

3. Stash operator watches for `BackupConfiguration` crd. Once, it found a `BackupConfiguration` crd, it creates a Backup triggering `CronJob` which creates `BackupSession` crd with CronJob schedule. 

4. Stash operator also watches for `BackupSession` crd. Once it found a `BackupSession` crd, it creates a Volume Snapshotter `Job`.

5. Then, Volume Snapshotter Job Creates a `VolumeSnapshot` vai kubernetes native API and Updates `BackupSession` status.

6. Finally CSI `external-snapshotter` controller watches for `VolumeSnapshot`. Once, it found a `VolumeSnapshot`, it tiggers `CreateSnapshot` operations againts the CSI endpoint and backup data in the Cloud Volume.


















         





