---
title: VolumeSnapshot Overview | Stash
description: An overview on how VolumeSnapshot works in Stash.
menu:
  product_stash_0.8.3:
    identifier: volume-snapshot-overview
    name: How VolumeSnapshot works?
    parent: volume-snapshot
    weight: 10
product_name: stash
menu_name: product_stash_0.8.3
section_menu_id: guides
---

# How VolumeSnapshot works in Stash

This guide will show you how Stash takes snapshot of PersistentVolumeClaims and restore them from snapshot using Kubernetes VolumeSnasphot API.

## Before You Begin

- At first, you need to have a Kubernetes cluster and ensure that a CSI driver that implements snapshots is deployed on your cluster. A list of CSI drivers that supports snapshots, You can learn from [here](https://kubernetes.io/blog/2019/01/17/update-on-volume-snapshot-alpha-for-kubernetes/). For this guide, we will use [GCE Persistent Disk CSI Driver](https://github.com/kubernetes-sigs/gcp-compute-persistent-disk-csi-driver).

- You need to enable the Kubernetes `VolumeSnapshotDataSource` alpha feature via Kubernetes feature gates
  - `--feature-gates=VolumeSnapshotDataSource=true`
- Install `Stash` in your cluster following the steps [here](https://appscode.com/products/stash/0.8.3/setup/install/).
- You should be familiar with the following Stash concepts:
  - [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md/)
  - [BackupSession](/docs/concepts/crds/backupsession.md/)
  - [RestoreSession](/docs/concepts/crds/restoresession.md/)
- You also should be familiar with the following Kubernetes concepts:
  - [VolumeSnapshot](https://kubernetes.io/docs/concepts/storage/volume-snapshots/#volumesnapshots)
  - [VolumeSnapshotContent](https://kubernetes.io/docs/concepts/storage/volume-snapshots/#volume-snapshot-contents)
  - [VolumeSnapshotClass](https://kubernetes.io/docs/concepts/storage/volume-snapshot-classes/)

## How Backup Works?

The following diagram shows how Stash creates VolumeSnapshot via Kubernetes native API. Open the image in a new tab to see the enlarged image.

<figure align="center">
  <img alt="Stash Backup Flow" src="/docs/images/guides/latest/volumesnapshot/volumesnapshot-overview.svg">
<figcaption align="center">Fig: Volume Snapshotting Process in Stash</figcaption>
</figure>

The `VolumeSnapshot` process consists of the following steps:

1. At first, a user creates a `BackupConfiguration` crd which specifies the targeted workload or targeted PVC.

2. Stash operator watches for `BackupConfiguration` crd.

3. Once it found a `BackupConfiguration` crd, it creates a `CronJob` to take a periodic backup of the target volumes.

4. The `CronJob` triggers backup on each schedule by creating a `BackupSession` crd.

5. Stash operator watches for `BackupSession` crd.

6. Once it found a `BackupSession` crd, it creates a volume snapshotter `Job` to take snapshot of the targeted volumes.

7. The volume snapshotter `Job` creates `VolumeSnapshot` crd for each PVC of the target and waits for the CSI driver to complete snapshotting. These `VolumeSnasphot` crd names follow the following format:

```console
  <PVC name>-<BackupSession creation timestamp in Unix epoch seconds>
```

8. CSI `external-snapshotter` controller watches for `VolumeSnapshot`.

9. Once it found a `VolumeSnapshot` object, it backups `VolumeSnapshot` in the respective cloud storage.

10. Once the snapsotting is completed, Stash Operator updates the `status.phase` field of the `BackupSession` crd.

## How Restore Works?

The following diagram shows how Stash restores PersistentVolumeClaims from snapshot using Kubernetes VolumeSnapshot API. Open the image in a new tab to see the enlarged image.

<figure align="center">
  <img alt="Stash Backup Flow" src="/docs/images/guides/latest/volumesnapshot/restore-overview.svg">
<figcaption align="center">Fig: Restore process from snapshot in Stash</figcaption>
</figure>

The restore process consists of the following steps:

1. At first, a user creates a `RestoreSession` crd which specifies the `volumeClaimTemplates`. `volumeClaimTemplates` holds the information about `VolumeSnapshot`.

2. Stash operator watches for `RestoreSession` crd.

3. Once it found a `RestoreSession` crd, it creates a Restore `Job`to restore PVC from the snapshot

4. The restore `Job` creates PVC with `spec.dataSource` field set to the respective VolumeSnapshot name.

5. CSI `external-snapshotter` controller watches for PVC.

6. Once it found a new PVC with `spec.dataSource` field set, it reads the information about the `VolumeSnapshot`.

7. The controller downloads the respective data from the cloud and populate the PVC with it.

8. Once restore process is completed, the Stash operator updates the `status.phase` field of the `BackupSession` crd.

## Next Steps

1. See a step by step guide to snapshot a Deployment's volumes from [here](/docs/guides/latest/volumesnapshot/deployment.md).

2. See a step by step guide to snapshot a StatefulSet's volumes from [here](/docs/guides/latest/volumesnapshot/statefulset.md).

3. See a step by step guide to snapshot a stand-alone PVC from [here](/docs/guides/latest/volumesnapshot/standalonepvc.md).
