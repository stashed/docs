# Volume Snapshots

## Introduction

Kubernetes v1.12 introduces alpha support for volume snapshotting. This feature allows creating/deleting volume snapshots, and the ability to create new volumes from a snapshot natively using the Kubernetes API.

Volume Snapshots are only supported for CSI drivers. Kubernetes CSI currently enables CSI drivers to expose the following functionality via the Kubernetes API:

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

Before using kubernetes volume snapshotting, you must:

* Ensure a CSI driver implementing snapshots is deployed and running on your Kubernetes cluster.
* Enable the Kubernetes Volume Snapshotting feature via new Kubernetes feature gate (disabled by default for alpha):
    * Set the following flag on the API server binary:  `--feature-gates=VolumeSnapshotDataSource=true`

## Volume Snapshot using Stash

This Section will show you how to use Stash for creating or restoring Volume Snapshots via kubernetes native API. Here we are going to create or restore PVC data. 

#### Requirements
At first, you need to have a kubernetes cluster and ensure that a CSI driver that implements snapshots is deployed on your cluster. For testing perpose, we will use [GCE Persistent Disk CSI Driver](https://github.com/kubernetes-sigs/gcp-compute-persistent-disk-csi-driver).Then `kubectl` command-line tool must be configured to communicate with your cluster.

* You need to enable the Kubernetes Volume Snapshotting feature via new Kubernetes feature gate 
    * `--feature-gates=VolumeSnapshotDataSource=true`
* Install `Stash` in your cluster following the steps [here](https://appscode.com/products/stash/0.8.3/setup/install/)
* You should be familiar with the following Stash concepts:
    * [BackupConfiguration]()
    * [BackupSession]()
    * [VolumeSnapshotContent](https://kubernetes.io/docs/concepts/storage/volume-snapshots/#volume-snapshot-contents)
    * [VolumeSnapshot](https://kubernetes.io/docs/concepts/storage/volume-snapshots/#volumesnapshots)
    * [VolumeSnapshotClass](https://kubernetes.io/docs/concepts/storage/volume-snapshot-classes/)

To keep things isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$kubectl create ns demo
namespace/demo created
```
>Note: YAML files used in this tutorial are stored in [/docs/examples/volume-snapshot](/docs/examples/volume-snapshot) directory of [appscode/stash](https://github.com/stashed/stash) repository.

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

6. Finally CSI `external-snapshotter` watches for `VolumeSnapshot`. Once, it found a `VolumeSnapshot`, it tiggers `CreateSnapshot` operations againts the CSI endpoint and backup Snapshot in the Cloud Volume.

#### Volume Snapshot

In order to create Volume Snapshot, At first, We will create a PVC that holds some sample data.

Below, The YAML for the PVC we are going to create.

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: backup-pvc
  namespace: demo
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 6Gi

```
Before Creating PVC, we need [StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/#the-storageclass-resource). A claim can request a particular class by specifying the name of a `StorageClass` using the attribute `storageClassName`. Only PVs of the requested class, ones with the same `storageClassName` as the PVC, can be bound to the PVC.

Below, The YAML for the StorageClass we are going to create

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
parameters:
  type: pd-standard
provisioner: pd.csi.storage.gke.io
reclaimPolicy: Delete
volumeBindingMode: Immediate
```
The [volumeBindingMode](https://kubernetes.io/docs/concepts/storage/storage-classes/#volume-binding-mode) field controls when volume binding and dynamic provisioning should occur. By default, the `Immediate` mode indicates that volume binding and dynamic provisioning occurs once the PVC is created and `WaitForFirstConsumer` mode indicates that volume binding and provisioning occurs until a pod using the PVC is created.

Lets create the StorageClass and PVC respectively we have shown above.

```console
kubectl apply -f ./docs/examples/volume-snapshot/storageclass.yaml
storageclass.storage.k8s.io/standard created
```
```console
kubectl apply -f ./docs/examples/volume-snapshot/pvc.yaml
persistentvolumeclaim/backup-pvc created
```
###### Deploy Workload:

Now, deploy the following Deployment. Here, we have mounted the PVC to workload.

Below, the YAML for the Deployment we are going to create.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: stash-demo
  name: stash-demo
  namespace: demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: stash-demo
  template:
    metadata:
      labels:
        app: stash-demo
      name: busybox
    spec:
      containers:
      - args: ["touch source/data/sample-file.txt && sleep 3000"]
        command: ["/bin/sh", "-c"]
        image: busybox
        imagePullPolicy: IfNotPresent
        name: busybox
        volumeMounts:
        - mountPath: /source/data
          name: source-data
      restartPolicy: Always
      volumes:
      - name: source-data
        persistentVolumeClaim:
         claimName: backup-pvc
```
















         





