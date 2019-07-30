---
title: Snapshot StatefulSet Volumes | Stash
description: An step by step guide showing how to snapshot the volumes of a StatefulSet
menu:
  product_stash_0.8.3:
    identifier: volume-snapshot-statefulset
    name: Snapshot StatefulSet Volumes
    parent: volume-snapshot
    weight: 30
product_name: stash
menu_name: product_stash_0.8.3
section_menu_id: guides
---

# Snapshotting the volumes of a StatefulSet

This guide will show you how to use Stash to snapshot the volumes of a StatefulSets and restore them from snapshot using Kubernetes [VolumeSnapshot](https://kubernetes.io/docs/concepts/storage/volume-snapshots/) API. In this guide, we are going to backup the volumes in Google Cloud Platform with the help of [GCE Persistent Disk CSI Driver](https://github.com/kubernetes-sigs/gcp-compute-persistent-disk-csi-driver).

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
- `snapshotter` field to point to the respective CSI driver that is responsible for taking snapshot. As we are using [GCE Persistent Disk CSI Driver](https://github.com/kubernetes-sigs/gcp-compute-persistent-disk-csi-driver), we are going to set `pd.csi.storage.gke.io` to this field.

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

>Note: YAML files used in this tutorial are stored in [/docs/examples/guides/latest/volumesnapshot](/docs/examples/guides/latest/volumesnapshot) directory of [stashed/docs](https://github.com/stashed/docs) repository.

## Take Volume Snapshot

When you create a Statefulset, there is no need to create PVCs separately, because all replicas in Statefulset use different PVCs to store data. Kubernetes allows us to define a `volumeClaimTemplates` in Statefulset so that new PVC is created for each replica automatically. We are going to take snapshot of those PVCs using Stash.

**Deploy StatefulSet :**

Now, we are going to deploy a Statefulset. This Statefulset will automatically generate sample data in `/source/data` directory.

Below is the YAML of the Statefulset that we are going to create,

```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc
  labels:
    app: demo
  namespace: demo
spec:
  ports:
    - port: 80
      name: web
  clusterIP: None
  selector:
    app: stash
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: stash-demo
  namespace: demo
spec:
  selector:
    matchLabels:
      app: stash
  serviceName: svc
  replicas: 3
  template:
    metadata:
      labels:
        app: stash
    spec:
      containers:
        - args: ["echo $(POD_NAME) > /source/data/data.txt && sleep 3000"]
          command: ["/bin/sh", "-c"]
          env:
            - name:  POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath:  metadata.name
          name: nginx
          image: nginx
          ports:
            - containerPort: 80
              name: web
          volumeMounts:
            - name: source-data
              mountPath: /source/data
  volumeClaimTemplates:
    - metadata:
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

Let's create the Statefulset we have shown above.

```console
$ kubectl create -f ./docs/examples/guides/latest/volumesnapshot/statefulset/statefulset.yaml
service/svc created
statefulset.apps/stash-demo created
```

Now, wait for the pod of Statefulset to go into the `Running` state.

```console
$ kubectl get pod -n demo
NAME           READY   STATUS    RESTARTS   AGE
stash-demo-0   1/1     Running   0          97s
stash-demo-1   1/1     Running   0          67s
stash-demo-2   1/1     Running   0          39s
```

Let's find out the PVCs created for these replicas,

```console
kubectl get pvc -n demo
NAME                       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
source-data-stash-demo-0   Bound    pvc-760c1734-a6cc-11e9-9f3a-42010a800050   1Gi        RWO            standard       70s
source-data-stash-demo-1   Bound    pvc-86f5b3bd-a6cc-11e9-9f3a-42010a800050   1Gi        RWO            standard       42s
source-data-stash-demo-2   Bound    pvc-9c9f542f-a6cc-11e9-9f3a-42010a800050   1Gi        RWO            standard       5s
```

Verify that the sample data has been created in `/source/data` directory using the following command,

```console
$ kubectl exec -n demo stash-demo-0 -- cat /source/data/data.txt
stash-demo-0
$ kubectl exec -n demo stash-demo-1 -- cat /source/data/data.txt
stash-demo-1
$ kubectl exec -n demo stash-demo-2 -- cat /source/data/data.txt
stash-demo-2
```

**Create BackupConfiguration :**

Now, create a `BackupConfiguration` crd to take snapshot of the PVCs of `stash-demo` Statefulset.

Below is the YAML of the `BackupConfiguration` that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: statefulset-volume-snapshot
  namespace: demo
spec:
  schedule: "*/1 * * * *"
  driver: VolumeSnapshotter
  target:
    ref:
      apiVersion: apps/v1
      kind: StatefulSet
      name: stash-demo
 #    replicas : 1
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
$ kubectl apply -f ./docs/examples/guides/latest/volumesnapshot/statefulset/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/statefulset-volume-snapshot created
```

**Verify CronJob :**

If everything goes well, Stash will create a `CronJob` to take periodic snapshot of `stash-demo-0` , `stash-demo-1` and `stash-demo-2` volumes of the Statefulset with the schedule specified in `spec.schedule` field of `BackupConfiguration` crd.

 Check that the `CronJob` has been created using the following command,

```console
$ kubectl get cronjob -n demo
NAME                          SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
statefulset-volume-snapshot   */1 * * * *   False     0        <none>          18s
```

**Wait for BackupSession :**

The `statefulset-volume-snapshot` CronJob will trigger a backup on each schedule by creating a `BackpSession` crd.

Wait for the next schedule for backup. Run the following command to watch `BackupSession` crd,

```console
$ watch -n 1 kubectl get backupsession -n demo
Every 1.0s: kubectl get backupsession -n demo                      suaas-appscode: Tue Jun 18 18:35:41 2019

NAME                                     BACKUP-CONFIGURATION           PHASE        AGE
statefulset-volume-snapshot-1563177551   statefulset-volume-snapshot   Running      57s
statefulset-volume-snapshot-1563177551   statefulset-volume-snapshot   Succeeded    57s
```

We can see above that the backup session has succeeded. Now, we are going to verify that the VolumeSnapshot has been created and the snapshots has been stored in the respective backend.

**Verify Volume Snapshot :**

Once a `BackupSession` crd is created, it creates volume snapshotter `Job`. Then the `Job` creates `VolumeSnapshot` crd for the targeted PVC.The `VolumeSnapshot` name follows the following pattern:

```console
 <PVC claim name>-<statefulset name>-<pod ordinal>-<backup session creation timestamp in Unix epoch seconds>
```

Check that the `VolumeSnapshot` has been created Successfully.

```console
$ kubectl get volumesnapshot -n demo
NAME                                  AGE
source-data-stash-demo-0-1563177551   115s
source-data-stash-demo-1-1563177551   115s
source-data-stash-demo-2-1563177551   115s
```

Let's find out the actual snapshot name that will be saved in the Google Cloud by the following command,

```console
kubectl get volumesnapshot source-data-stash-demo-0-1563177551 -n demo -o yaml
```

```yaml
apiVersion: snapshot.storage.k8s.io/v1alpha1
kind: VolumeSnapshot
metadata:
  creationTimestamp: "2019-07-15T07:59:13Z"
  finalizers:
  - snapshot.storage.kubernetes.io/volumesnapshot-protection
  generation: 4
  name: source-data-stash-demo-0-1563177551
  namespace: demo
  resourceVersion: "18764"
  selfLink: /apis/snapshot.storage.k8s.io/v1alpha1/namespaces/demo/volumesnapshots/source-data-stash-demo-0-1563177551
  uid: 6f3b49a9-a6d6-11e9-9f3a-42010a800050
spec:
  snapshotClassName: default-snapshot-class
  snapshotContentName: snapcontent-6f3b49a9-a6d6-11e9-9f3a-42010a800050
  source:
    apiGroup: null
    kind: PersistentVolumeClaim
    name: source-data-stash-demo-0
status:
  creationTime: "2019-07-15T07:59:14Z"
  readyToUse: true
  restoreSize: 1Gi
```

Here, `spec.snapshotContentName` field specifies the name of the `VolumeSnapshotContent` crd. It also represents the actual snapshot name that has been saved in Google Cloud. If we navigate to the `Snapshots` tab in the GCP console, we should see snapshot `snapcontent-6f3b49a9-a6d6-11e9-9f3a-42010a800050` has been stored successfully.

<figure align="center">
  <img alt="Stash Backup Flow" src="/docs/images/guides/latest/volumesnapshot/statefulset.png">
<figcaption align="center">Fig: Snapshots in GCE Bucket</figcaption>
</figure>

## Restore PVC from VolumeSnapshot

This section will show you how to restore PVCs from the snapshots we have taken in the earlier section.

**Create RestoreSession :**

At first, we have to create a `RestoreSession` crd to restore PVCs from respective the snapshots.

Below is the YAML of the `RestoreSesion` crd that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: restore-pvc
  namespace: demo
spec:
  driver: VolumeSnapshotter
  target:
    replicas : 3
    volumeClaimTemplates:
      - metadata:
          name: restore-data-restore-demo-${POD_ORDINAL}
        spec:
          accessModes: [ "ReadWriteOnce" ]
          storageClassName: "standard"
          resources:
            requests:
              storage: 1Gi
          dataSource:
            kind: VolumeSnapshot
            name: source-data-stash-demo-${POD_ORDINAL}-1563177551
#           name: source-data-stash-demo-0-1563177551
            apiGroup: snapshot.storage.k8s.io
```

Here,

- `spec.target.replicas`: `spec.target.replicas` specify the number of replicas of a StatefulSet whose volumes were backed up and Stash uses this field to dynamically create the desired number of PVCs and initialize them from respective or Specific VolumeSnapShots.
- `spec.target.volumeClaimTemplates`:
  - `metadata.name` is a template for the name of the restored PVC that will be created by Stash. You have to provide this named template to match with the desired PVC of a StatefulSet. For example, if you want to deploy a StatefulSet named `stash-demo` with `volumeClaimTemplate` name `my-volume`, the PVCs of your StatefulSet will be `my-volume-stash-demo-0`, `my-volume-stash-demo-1` and so on. In this case, you have to provide `volumeClaimTemplate` name in `RestoreSession` in the following format:

    ```console
    <volume claim name>-<statefulset name>-${POD_ORDINAL}
    ```

    So for the above example, `volumeClaimTemplate` name for `RestoreSession` will be `my-volume-stash-demo-${POD_ORDINAL}`.
  - `spec.dataSource`: `spec.dataSource` specifies the source of the data from where the newly created PVC will be initialized. It requires following fields to be set:
    - `apiGroup` is the group for resource being referenced. Now, Kubernetes supports only `snapshot.storage.k8s.io`.
    - `kind` is resource of the kind being referenced. Now, Kubernetes supports only `VolumeSnapshot`.
    - `name` is the `VolumeSnapshot` resource name. In `RestoreSession` crd, You must provide the name in the following format:

      ```console
      <VolumeSnapshot name prefix>-${POD_ORDINAL}-<timestamp in Unix  epoch seconds>
      ```

      The `${POD_ORDINAL}` variable is resolved by Stash. If you don't provide this variable and specify ordinal manually, all the PVC will be restored from the same VolumeSnapshot.

Let's create the `RestoreSession` crd we have shown above.

```console
$ kubectl create -f ./docs/examples/guides/latest/volumesnapshot/statefulset/restoresession.yaml
restoresession.stash.appscode.com/restore-pvc created
```

Once, you have created the `RestoreSession` crd, Stash will create a job to restore. We can watch the `RestoreSession` phase to check if the restore process has succeeded or not.

Run the following command to watch RestoreSession phase,

```console
$ watch -n 1 kubectl get restore -n demo
Every 1.0s: kubectl get restore -n demo                      suaas-appscode: Tue Jun 18 18:35:41 2019

NAME          REPOSITORY-NAME   PHASE       AGE
restore-pvc                     Running     10s
restore-pvc                     Succeeded   1m
```

So, we can see from the output of the above command that the restore process succeeded.

**Verify Restored PVC :**

Once the restore process is complete, we are going to see that new PVCs with the name `restore-data-restore-demo-0` , `restore-data-restore-demo-1` and `restore-data-restore-demo-2` has been created.

Verify that the PVCs has been created by the following command,

```console
$ kubectl get pvc -n demo
NAME                          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
restore-data-restore-demo-0   Bound    pvc-ed35c54d-a6dc-11e9-9f3a-42010a800050   1Gi        RWO            standard       13s
restore-data-restore-demo-1   Bound    pvc-ed3bcb82-a6dc-11e9-9f3a-42010a800050   1Gi        RWO            standard       13s
restore-data-restore-demo-2   Bound    pvc-ed3fed79-a6dc-11e9-9f3a-42010a800050   1Gi        RWO            standard       13s
```

Notice the `STATUS` field. It indicates that the respective PV has been provisioned and initialized from the respective VolumeSnapshot by CSI driver and the PVC has been bound with the PV.

>The [volumeBindingMode](https://kubernetes.io/docs/concepts/storage/storage-classes/#volume-binding-mode) field controls when volume binding and dynamic provisioning should occur. Kubernetes allows `Immediate` and `WaitForFirstConsumer` modes for binding volumes. The `Immediate` mode indicates that volume binding and dynamic provisioning occurs once the PVC is created and `WaitForFirstConsumer` mode indicates that volume binding and provisioning does not occur until a pod is created that uses this PVC. By default `volumeBindingMode` is `Immediate`.

>If you use `volumeBindingMode: WaitForFirstConsumer`, respective PVC will be initialized from respective VolumeSnapshot after you create a workload with that PVC. In this case, Stash will mark the restore session as completed with phase `Unknown`.

**Verify Restored Data :**

We are going to create a new Statefulset with the restored PVCs to verify whether the backed up data has been restored.

Below, the YAML for the Statefulset we are going to create.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: restore-svc
  labels:
    app: restore-demo
  namespace: demo
spec:
  ports:
    - port: 80
      name: web
  clusterIP: None
  selector:
    app: restore-demo
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: restore-demo
  namespace: demo
spec:
  selector:
    matchLabels:
      app: restore-demo
  serviceName: svc
  replicas: 3
  template:
    metadata:
      labels:
        app: restore-demo
    spec:
      containers:
        - args:
            - sleep
            - "3600"
          name: nginx
          image: nginx
          ports:
            - containerPort: 80
              name: web
          volumeMounts:
            - name: restore-data
              mountPath: /restore/data
  volumeClaimTemplates:
    - metadata:
        name: restore-data
        namespace: demo
      spec:
        accessModes:
          - ReadWriteOnce
        storageClassName: standard
        resources:
          requests:
            storage: 1Gi
```

Let's create the Statefulset we have shown above.

```console
$ kubectl create -f ./docs/examples/guides/latest/volumesnapshot/statefulset/restored-statefulset.yaml
service/svc created
statefulset.apps/restore-demo created
```

Now, wait for the pod of the Statefulset to go into the `Running` state.

```console
$ kubectl get pod -n demo
NAME             READY   STATUS    RESTARTS   AGE
restore-demo-0   1/1     Running   0          65s
restore-demo-1   1/1     Running   0          46s
restore-demo-2   1/1     Running   0          26s
```

Verify that the backed up data has been restored in `/restore/data` directory using the following command,

```console
$ kubectl exec -n demo restore-demo-0 -- cat /restore/data/data.txt
stash-demo-0
$ kubectl exec -n demo restore-demo-1 -- cat /restore/data/data.txt
stash-demo-1
$ kubectl exec -n demo restore-demo-2 -- cat /restore/data/data.txt
stash-demo-2
```

## Advance Use-Case

Stash can also backup only single replica or restore same data on all replicas of a StatefulSet. This is particularly useful when all replicas of the StatefulSet contains same data. For example, in [MongoDB ReplicaSet](https://kubedb.com/docs/0.12.0/guides/mongodb/clustering/replication_concept/) all the pod contains same data. In this case backup only single replica is enough. Similarly, it might be useful in some cases where all the replicas need to be initialized with same data.

### Backup only Single Replica

This section will show you how to snapshot only a single replica of a Statefulset volume.

**Create BackupConfiguration :**

Now, create a `BackupConfiguration` crd to take snapshot of a single PVC of the `stash-demo` Statefulset.

Below is the YAML of the `BackupConfiguration` that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: statefulset-volume-snapshot
  namespace: demo
spec:
  schedule: "*/1 * * * *"
  driver: VolumeSnapshotter
  target:
    ref:
      apiVersion: apps/v1
      kind: StatefulSet
      name: stash-demo
      replicas : 1
    snapshotClassName: default-snapshot-class
  retentionPolicy:
    name: 'keep-last-5'
    keepLast: 5
    prune: true
```

Here,

- `spec.replicas` specifies the number of replicas (starting from 0th) whose data should be backed up. If it is set to 1, Stash will take snapshot only the volumes of `<claim-name>-<statefulset-name>-0` pod. For replica set to 2, Stash will take snapshot only the volumes of `<claim-name>-<statefulset-name>-0` and `<claim-name>-<statefulset-name>-1` pods and so on.

Let's create the `BackupConfiguration` crd we have shown above.

```console
$ kubectl apply -f ./docs/examples/guides/latest/volumesnapshot/statefulset/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/statefulset-volume-snapshot created
```

**Verify CronJob :**

If everything goes well, Stash will create a `CronJob` to take periodic snapshot of `stash-demo-0` volume of the Statefulset with the schedule specified in `spec.schedule` field of `BackupConfiguration` crd.

 Check that the `CronJob` has been created using the following command,

```console
$ kubectl get cronjob -n demo
NAME                          SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
statefulset-volume-snapshot   */1 * * * *   False     0        <none>          18s
```

**Wait for BackupSession :**

The `statefulset-volume-snapshot` CronJob will trigger a backup on each schedule by creating a `BackpSession` crd.

Wait for the next schedule for backup. Run the following command to watch `BackupSession` crd,

```console
$ watch -n 1 kubectl get backupsession -n demo
Every 1.0s: kubectl get backupsession -n demo                      suaas-appscode: Tue Jun 18 18:35:41 2019

NAME                                     BACKUPCONFIGURATION           PHASE        AGE
statefulset-volume-snapshot-1563181264   statefulset-volume-snapshot   Running      57s
statefulset-volume-snapshot-1563181264   statefulset-volume-snapshot   Succeeded    57s
```

We can see above that the backup session has succeeded. Now, we are going to verify that the VolumeSnapshot has been created and the snapshot has been stored in the respective backend.

**Verify Volume Snapshotting and Backup :**

Once a `BackupSession` crd is created, Stash creates a volume snapshotter `Job`. Then the `Job` creates `VolumeSnapshot` crd for the targeted PVC. The `VolumeSnapshot` name follows the following pattern:

```console
 <PVC claim name>-<backup session creation timestamp in Unix epoch seconds>
```

Check that the `VolumeSnapshot` has been created Successfully.

```console
$ kubectl get volumesnapshot -n demo
NAME                                  AGE
source-data-stash-demo-0-1563181264   67s
```

Let's find out the actual snapshot name that will be saved in the GCP by the following command,

```console
kubectl get volumesnapshot source-data-stash-demo-0-1563181264 -n demo -o yaml
```

```yaml
apiVersion: snapshot.storage.k8s.io/v1alpha1
kind: VolumeSnapshot
metadata:
  creationTimestamp: "2019-07-15T09:01:06Z"
  finalizers:
  - snapshot.storage.kubernetes.io/volumesnapshot-protection
  generation: 4
  name: source-data-stash-demo-0-1563181264
  namespace: demo
  resourceVersion: "24310"
  selfLink: /apis/snapshot.storage.k8s.io/v1alpha1/namespaces/demo/volumesnapshots/source-data-stash-demo-0-1563181264
  uid: 14984cd3-a6df-11e9-9f3a-42010a800050
spec:
  snapshotClassName: default-snapshot-class
  snapshotContentName: snapcontent-14984cd3-a6df-11e9-9f3a-42010a800050
  source:
    apiGroup: null
    kind: PersistentVolumeClaim
    name: source-data-stash-demo-0
status:
  creationTime: "2019-07-15T09:01:07Z"
  readyToUse: true
  restoreSize: 1Gi
```

Here, `spec.snapshotContentName` field specifies the name of the `VolumeSnapshotContent` crd. It also represents the actual snapshot name that has been saved in GCP. If we navigate to the `Snapshots` tab in the GCP console, we should see the snapshot `snapcontent-14984cd3-a6df-11e9-9f3a-42010a800050` has been stored successfully.

<figure align="center">
  <img alt="Stash Backup Flow" src="/docs/images/guides/latest/volumesnapshot/statefulset2.png">
<figcaption align="center">Fig: Snapshot in GCE Bucket</figcaption>
</figure>

### Restore Same Data in all Replicas

This section will show you how to restore PVCs from the snapshot that  we have taken in the earlier section.

**Create RestoreSession :**

At first, we have to create a `RestoreSession` crd to restore PVCs from the respective snapshot.

Below is the YAML of the `RestoreSesion` crd that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: restore-pvc
  namespace: demo
spec:
  driver: VolumeSnapshotter
  target:
    replicas : 3
    volumeClaimTemplates:
      - metadata:
          name: restore-data-restore-demo-${POD_ORDINAL}
        spec:
          accessModes: [ "ReadWriteOnce" ]
          storageClassName: "standard"
          resources:
            requests:
              storage: 1Gi
          dataSource:
            kind: VolumeSnapshot
            name: source-data-stash-demo-0-1563181264
#            name: source-data-stash-demo-${POD_ORDINAL}-1563177551
            apiGroup: snapshot.storage.k8s.io
```

Here,

- `spec.dataSource.name`: `spec.dataSource.name` is the `VolumeSnapshot` resource name. data will be restored in all replica from single VolumeSnapshot.

Let's create the `BackupConfiguration` crd we have shown above.

```console
$ kubectl create -f ./docs/examples/guides/latest/volumesnapshot/statefulset/restoresession.yaml
restoresession.stash.appscode.com/restore-pvc created
```

Once, you have created the `RestoreSession` crd, Stash will create a job to restore. We can watch the `RestoreSession` phase to check if the restore process has succeeded or not.

Run the following command to watch RestoreSession phase,

```console
$ watch -n 1 kubectl get restore -n demo
Every 1.0s: kubectl get restore -n demo                      suaas-appscode: Tue Jun 18 18:35:41 2019

NAME          REPOSITORY-NAME   PHASE       AGE
restore-pvc                     Running     10s
restore-pvc                     Succeeded   1m
```

So, we can see from the output of the above command that the restore process succeeded.

**Verify Restored PVC :**

Once the restore process is complete, we are going to see that new PVCs with the name `restore-data-restore-demo-0` , `restore-data-restore-demo-1` and `restore-data-restore-demo-2` have been created successfully.

check that the status of the PVCs are bound,

```console
$ kubectl get pvc -n demo
NAME                          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
restore-data-restore-demo-0   Bound    pvc-745e0f51-a6e0-11e9-9f3a-42010a800050   1Gi        RWO            standard       5m23s
restore-data-restore-demo-1   Bound    pvc-746227e7-a6e0-11e9-9f3a-42010a800050   1Gi        RWO            standard       5m23s
restore-data-restore-demo-2   Bound    pvc-74674656-a6e0-11e9-9f3a-42010a800050   1Gi        RWO            standard       5m23s
```

**Verify Restored Data :**

We are going to create a new Statefulset to verify whether the restored data has been restored successfully.

Below, the YAML for the Statefulset we are going to create.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: restore-svc
  labels:
    app: restore-demo
  namespace: demo
spec:
  ports:
    - port: 80
      name: web
  clusterIP: None
  selector:
    app: restore-demo
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: restore-demo
  namespace: demo
spec:
  selector:
    matchLabels:
      app: restore-demo
  serviceName: svc
  replicas: 3
  template:
    metadata:
      labels:
        app: restore-demo
    spec:
      containers:
        - args:
            - sleep
            - "3600"
          name: nginx
          image: nginx
          ports:
            - containerPort: 80
              name: web
          volumeMounts:
            - name: restore-data
              mountPath: /restore/data
  volumeClaimTemplates:
    - metadata:
        name: restore-data
        namespace: demo
      spec:
        accessModes:
          - ReadWriteOnce
        storageClassName: standard
        resources:
          requests:
            storage: 1Gi
```

Let's create the Statefulset we have shown above.

```console
$ kubectl create -f ./docs/examples/guides/latest/volumesnapshot/statefulset/restored-statefulset.yaml
service/svc created
statefulset.apps/restore-demo created
```

Now, wait for the pod of Statefulset to go into the `Running` state.

```console
$ kubectl get pod -n demo
NAME             READY   STATUS    RESTARTS   AGE
restore-demo-0   1/1     Running   0          3m9s
restore-demo-1   1/1     Running   0          2m50s
restore-demo-2   1/1     Running   0          2m30s
```

Verify that the backed up data has been restored in `/restore/data` directory using the following command,

```console
$ kubectl exec -n demo restore-demo-0 -- cat /restore/data/data.txt
stash-demo-0
$ kubectl exec -n demo restore-demo-1 -- cat /restore/data/data.txt
stash-demo-0
$ kubectl exec -n demo restore-demo-2 -- cat /restore/data/data.txt
stash-demo-0
```

## Cleaning Up

To clean up the Kubernetes resources created by this tutorial, run:

```console
kubectl delete -n demo statefulset stash-demo
kubectl delete -n demo statefulset restore-demo
kubectl delete -n demo backupconfiguration statefulset-volume-snapshot
kubectl delete -n demo restoresession restore-pvc
kubectl delete -n demo storageclass standard
kubectl delete -n demo volumesnapshotclass default-snapshot-class
```
