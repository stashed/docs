---
title: Snapshotting PVCs | Stash
description: An step by step guide showing how to snapshot a stand-alone PVC
menu:
  product_stash_0.8.3:
    identifier: volume-snapshot-pvc
    name: Snapshotting PVCs
    parent: volume-snapshot
    weight: 40
product_name: stash
menu_name: product_stash_0.8.3
section_menu_id: guides
---

# Snapshotting a Standalone PVC

This guide will show you how to use Stash to snapshot standalone PersistentVolumeClaims and restore that from snapshot using Kubernetes [VolumeSnapshot](https://kubernetes.io/docs/concepts/storage/volume-snapshots/) API. In this guide, we are going to backup the volumes in Google Cloud Platform with the help of [GCE Persistent Disk CSI Driver](https://github.com/kubernetes-sigs/gcp-compute-persistent-disk-csi-driver).

## Before You Begin

- At first, you need to be familiar with the [GCE Persistent Disk CSI Driver](https://github.com/kubernetes-sigs/gcp-compute-persistent-disk-csi-driver).
- You need to enable the Kubernetes `VolumeSnapshotDataSource` alpha feature via Kubernetes feature gates
  - `--feature-gates=VolumeSnapshotDataSource=true`
- Install `Stash` in your cluster following the steps [here](https://appscode.com/products/stash/0.8.3/setup/install/).
- If you don't know how VolumeSnapshot works in Stash, please visit [here](/docs/guides/latest/volumesnapshot/overview.md).

## Prepare for VolumeSnapshot

If you don't already have a StorageClass that uses the CSI driver that supports VolumeSnapshot feature, create one first. Here, we are going to create `StorageClass` that uses [GCE Persistent Disk CSI Driver](https://github.com/kubernetes-sigs/gcp-compute-persistent-disk-csi-driver).

Sample `StorageClass` YAML are given below,

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

Let's create the `StorageClass` we have shown above,

```console
$ kubectl apply -f ./docs/examples/guides/latest/volumesnapshot/storageclass.yaml
storageclass.storage.k8s.io/standard created
```

We also need a `VolumeSnapshotClass`. We are going to use the following `VolumeSnapshotClass` for this tutorial,

```yaml
apiVersion: snapshot.storage.k8s.io/v1alpha1
kind: VolumeSnapshotClass
metadata:
  annotations:
    snapshot.storage.kubernetes.io/is-default-class: "true"
  name: default-snapshot-class
snapshotter: pd.csi.storage.gke.io
```

Here,

- `metadata.annotations` annotations are used to set default [volumeSnapshotClass](https://kubernetes.io/blog/2018/10/09/introducing-volume-snapshot-alpha-for-kubernetes/).
- `snapshotter` field to point to the respective CSI driver that is responsible for taking snapshot. As we are using [GCE Persistent Disk CSI Driver](https://github.com/kubernetes-sigs/gcp-compute-persistent-disk-csi-driver), we are going to use `pd.csi.storage.gke.io` in this field.

Let's create the `volumeSnapshotClass` crd we have shown above,

```console
$ kubectl apply -f ./docs/examples/guides/latest/volumesnapshot/default-volumesnapshotclass.yaml
volumesnapshotclass.snapshot.storage.k8s.io/default-snapshot-class created
```

To keep everything isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$ kubectl create ns demo
namespace/demo created
```

>Note: YAML files used in this tutorial are stored in [/docs/examples/guides/latest/volumesnapshot](/docs/examples/guides/latest/volumesnapshot) directory of [stashed/stash](https://github.com/stashed/stash) repository.

## Take Volume Snapshot

Here, we are going to create a PVC and mount it with a pod and we will also generate some sample data on it. Then, we will take snapshot of this PVC using Stash.

**Create PersistentVolumeClaim :**

At first, let's create a PVC. We will mount this PVC in a pod.

Below is the YAML of the sample PVC,

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: source-data
  namespace: demo
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 1Gi
```

Let's create the PVC we have shown above.

```console
$ kubectl apply -f ./docs/examples/guides/latest/volumesnapshot/standalone-pvc/source-pvc.yaml
persistentvolumeclaim/source-data created
```

**Create Pod :**

Now, we will deploy a pod that uses the above PVC. This pod will automatically create `data.txt` file in `/source/data`  directory and write some sample data in it and also mounted the desired PVC in `/source/data`  directory.

Below is the YAML of the pod that we are going to create,

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: source-pod
  namespace: demo
spec:
  containers:
    - name: busybox
      image: busybox
      command: ["/bin/sh", "-c"]
      args: ["echo sample_data > /demo/data/data.txt && sleep 3000"]
      volumeMounts:
        - name: source-data
          mountPath: /source/data
  volumes:
    - name: source-data
      persistentVolumeClaim:
        claimName: source-data
        readOnly: false
```

Let's create the Pod we have shown above.

```console
$ kubectl apply -f ./docs/examples/guides/latest/volumesnapshot/standalone-pvc/source-pod.yaml
pod/source-pod created
```

Now, wait for the Pod to go into the `Running` state.

```console
$ kubectl get pod -n demo 
NAME         READY   STATUS    RESTARTS   AGE
source-pod   1/1     Running   0          25s
```

Verify that the sample data has been created in `/source/data` directory for `source-pod` pod
using the following command,

```console
$ kubectl exec -n demo source-pod -- cat /source/data/data.txt
sample_data
```

**Create BackupConfiguration :**

Now, create a `BackupConfiguration` crd to take snapshot of the `source-data` PVC.

Below is the YAML of the `BackupConfiguration` crd that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: pvc-volume-snapshot
  namespace: demo
spec:
  schedule: "*/1 * * * *"
  driver: VolumeSnapshotter
  target:
    ref:
      apiVersion: v1
      kind: PersistentVolumeClaim
      name: source-data
    snapshotClassName: default-snapshot-class
  retentionPolicy:
    name: 'keep-last-5'
    keepLast: 5
    prune: true
```

Here,

- `spec.schedule` is a [cron expression](https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/#schedule) indicates that `BackupSession` will be created at 1 minute interval.

- `spec.driver` indicates the name of the agent to use to back up the target. Currently, Stash supports `Restic`, `VolumeSnapshotter` drivers. The `VolumeSnapshotter` is used to backup/restore PVC using `VolumeSnapshot` API.

- `spec.target.ref`  refers to the backup target. `apiVersion`, `kind` and `name` refers to the `apiVersion`, `kind` and `name` of the targeted workload respectively. Stash will use this information to create a Volume Snapshotter Job for creating VolumeSnapshot.

- `spec.target.snapshotClassName` indicates the [VolumeSnapshotClass](https://kubernetes.io/docs/concepts/storage/volume-snapshot-classes/) to be used for volume snapshotting.

Let's create the `BackupConfiguration` crd we have shown above.

```console
$ kubectl apply -f ./docs/examples/guides/latest/volumesnapshot/standalone-pvc/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/pvc-volume-snapshot created
```

**Verify CronJob :**

If everything goes well, Stash will create a `CronJob` to take periodic snapshot of the PVC with the schedule specified in `spec.schedule` field of `BackupConfiguration` crd.

 Check that the `CronJob` has been created using the following command,

 ```console
 $ kubectl get cronjob -n demo 
NAME                          SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
pvc-volume-snapshot           */1 * * * *   False     0        39s             2m41s
 ```

**Wait for BackupSession :**

The `pvc-volume-snapshot` CronJob will trigger a backup on each scheduled time slot by creating a `BackpSession` crd.

Wait for a schedule to appear. Run the following command to watch `BackupSession` crd,

```console
$ watch -n 1 kubectl get backupsession -n demo
Every 1.0s: kubectl get backupsession -n demo                      suaas-appscode: Tue Jun 18 18:35:41 2019

NAME                                     BACKUPCONFIGURATION           PHASE       AGE
pvc-volume-snapshot-1563186667           pvc-volume-snapshot           Running     32s
pvc-volume-snapshot-1563186667           pvc-volume-snapshot           Succeeded   1m32s
```

We can see above that the backup session has succeeded. Now, we will verify that the `VolumeSnapshot` has been created and the snapshots has been stored in the respective backend.

**Verify Volume Snapshot :**

Once a `BackupSession` crd is created, it creates volume snapshotter `Job`. Then the `Job` creates `VolumeSnapshot` crd for the targeted PVC. The `VolumeSnapshot` name follows the following pattern:

```console
<PVC name>-<backup session creation timestamp in Unix epoch seconds>
```

Check that the `VolumeSnapshot` has been created Successfully.

```console
$ kubectl get volumesnapshot -n demo
NAME                     AGE
source-data-1563186667   1m30s
```

Let's find out the actual snapshot name that will be saved in the Google Cloud by the following command,

```console
kubectl get volumesnapshot source-data-1563186667  -n demo -o yaml
```

```yaml
apiVersion: snapshot.storage.k8s.io/v1alpha1
kind: VolumeSnapshot
metadata:
  creationTimestamp: "2019-07-15T10:31:09Z"
  finalizers:
  - snapshot.storage.kubernetes.io/volumesnapshot-protection
  generation: 4
  name: source-data-1563186667
  namespace: demo
  resourceVersion: "32098"
  selfLink: /apis/snapshot.storage.k8s.io/v1alpha1/namespaces/demo/volumesnapshots/source-data-1563186667
  uid: a8e8faeb-a6eb-11e9-9f3a-42010a800050
spec:
  snapshotClassName: default-snapshot-class
  snapshotContentName: snapcontent-a8e8faeb-a6eb-11e9-9f3a-42010a800050
  source:
    apiGroup: null
    kind: PersistentVolumeClaim
    name: source-data
status:
  creationTime: "2019-07-15T10:31:10Z"
  readyToUse: true
  restoreSize: 1Gi
```

Here, `spec.snapshotContentName` field specifies the name of the `VolumeSnapshotContent` crd. It also represents the actual snapshot name that has been saved in Google Cloud. If we navigate to the `Snapshots` tab in the GCP console, we will see snapshot `snapcontent-a8e8faeb-a6eb-11e9-9f3a-42010a800050` has been stored successfully.

<figure align="center">
  <img alt="Stash Backup Flow" src="/docs/images/guides/latest/volumesnapshot/standalone-pvc.png">
<figcaption align="center">Fig: Snapshots in GCE Bucket</figcaption>
</figure>

## Restore PVC from VolumeSnapshot

This section will show you how to restore the PVC from the snapshot we have taken in earlier section.

**Create RestoreSession :**

At first, we have to create a `RestoreSession` crd to restore the PVC from respective snapshot.

Below is the YAML of the RestoreSesion crd that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: restore-pvc
  namespace: demo
spec:
  driver: VolumeSnapshotter
  target:
    volumeClaimTemplates:
      - metadata:
          name: restore-data
        spec:
          accessModes: [ "ReadWriteOnce" ]
          storageClassName: "standard"
          resources:
            requests:
              storage: 1Gi
          dataSource:
            kind: VolumeSnapshot
            name: source-data-1563186667
            apiGroup: snapshot.storage.k8s.io
```

Here,

- `spec.target.volumeClaimTemplates`:
  - `metadata.name` is the name of the restored `PVC` or prefix of the `VolumeSnapshot` name.
  - `spec.dataSource`: `spec.dataSource` specifies the source of the data from where the newly created PVC will be initialized. It requires following fields to be set:
    - `apiGroup` is the group for resource being referenced. Now, Kubernetes supports only `snapshot.storage.k8s.io`.
    - `kind` is resource of the kind being referenced. Now, Kubernetes supports only `VolumeSnapshot`.
    - `name` is the `VolumeSnapshot` resource name. In `RestoreSession` crd, You must set the VolumeSnapshot name directly.

Let's create the `RestoreSession` crd we have shown above.

```console
$ kubectl create -f ./docs/examples/guides/latest/volumesnapshot/standalone-pvc/restoresession.yaml
restoresession.stash.appscode.com/restore-pvc created
```

Once, you have created the `RestoreSession` crd, Stash will create a job to restore. We can watch the `RestoreSession` phase to check if the restore process is succeeded or not.

Run the following command to watch RestoreSession phase,

```console
$ watch -n 1 kubectl get restore -n demo
Every 1.0s: kubectl get restore -n demo                      suaas-appscode: Tue Jun 18 19:32:35 2019

NAME          REPOSITORY-NAME   PHASE       AGE
restore-pvc                     Running     10s
restore-pvc                     Succeeded   1m
```

**Verify Restored PVC :**

Once a restore process is complete, we will see that new PVC with the name `restore-data` has been created.

To verify that the PVC has been created, run by the following command,

```console
$ kubectl get pvc -n demo
NAME           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
restore-data   Bound    pvc-c5f0e7f5-a6ec-11e9-9f3a-42010a800050   1Gi        RWO            standard       52s
```

Notice the `STATUS` field. It indicates that the respective PV has been provisioned and initialized from the respective VolumeSnapshot by CSI driver and the PVC has been bound with the PV.

>The [volumeBindingMode](https://kubernetes.io/docs/concepts/storage/storage-classes/#volume-binding-mode) field controls when volume binding and dynamic provisioning should occur. Kubernetes allows `Immediate` and `WaitForFirstConsumer` modes for binding volumes. The `Immediate` mode indicates that volume binding and dynamic provisioning occurs once the PVC is created and `WaitForFirstConsumer` mode indicates that volume binding and provisioning does not occur until a pod is created that uses this PVC. By default `volumeBindingMode` is `Immediate`.

>If you use `volumeBindingMode: WaitForFirstConsumer`, respective PVC will be initialized from respective VolumeSnapshot after you create a workload with that PVC. In this case, Stash will mark the restore session as completed with phase `Unknown`.

**Verify Restored Data :**

We will create a new pod with the restored PVC to verify whether the backed up data has been restored.

Below, the YAML for the Pod we are going to create.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: restored-pod
  namespace: demo
spec:
  containers:
    - name: busybox
      image: busybox
      args:
        - sleep
        - "3600"
      volumeMounts:
        - name: restore-data
          mountPath: /restore/data
  volumes:
    - name: restore-data
      persistentVolumeClaim:
        claimName: restore-data
        readOnly: false
```

Let's create the Pod we have shown above.

```console
$ kubectl apply -f ./docs/examples/guides/latest/volumesnapshot/standalone-pvc/restored-pod.yaml
pod/restored-pod created
```

Now, wait for the Pod to go into the `Running` state.

```console
$ kubectl get pod -n demo 
NAME           READY   STATUS    RESTARTS   AGE
restored-pod   1/1     Running   0          34s
```

Verify that the backed up data has been restored in `/restore/data` directory for `restored-pod` pod using the following command,

```console
$ kubectl exec -n demo restored-pod -- cat /restore/data/data.txt
sample_data
```

## Cleaning Up

To clean up the Kubernetes resources created by this tutorial, run:

```console
kubectl delete -n demo pod source-pod
kubectl delete -n demo pod restored-pod
kubectl delete -n demo backupconfiguration pvc-volume-snapshot
kubectl delete -n demo restoresession restore-pvc
kubectl delete -n demo storageclass standard
kubectl delete -n demo volumesnapshotclass default-snapshot-class
kubectl delete -n demo pvc --all
```
