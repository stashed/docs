## Snapshot Statefulste's Volumes

This guide will show you how to use Stash to snapshot Statefulset's volumes and restore them from snapshot using Kubernetes [VolumeSnapshot](https://kubernetes.io/docs/concepts/storage/volume-snapshots/) API.In this guide, we are going to backup the volumes in Google Cloud Storage with the help of [GCE Persistent Disk CSI Driver](https://github.com/kubernetes-sigs/gcp-compute-persistent-disk-csi-driver).

### Before You Begin

* At first, you need to be familiarize with the [GCE Persistent Disk CSI Driver](https://github.com/kubernetes-sigs/gcp-compute-persistent-disk-csi-driver).
* Also need to enable the Kubernetes `VolumeSnapshotDataSource` alpha feature via feature gates.
* You need to have Stash installed in your cluster. If you already don't have Stash installed, please follow this [steps](https://appscode.com/products/stash/0.8.3/setup/install/).
* If you don't know how VolumeSnapshot works in Stash, please visit [here](/docs/guides/latest/volumesnapshot/overview.md).

#### Prepare for VolumeSnapshot

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
The [volumeBindingMode](https://kubernetes.io/docs/concepts/storage/storage-classes/#volume-binding-mode) field controls when volume binding and dynamic provisioning should occur. Kubernetes allows `Immediate` and `WaitForFirstConsumer` modes for binding volumes. The `Immediate` mode indicates that volume binding and dynamic provisioning occurs once the PVC is created and `WaitForFirstConsumer` mode indicates that volume binding and provisioning does not occur until a pod is created that uses this PVC.

Let's create the `StorageClass` we have shown above,

```console
$ kubectl apply -f ./docs/examples/volume-snapshot/storageclass.yaml
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

* `metadata.annotations` annotations are used to set default [volumeSnapshotClass](https://kubernetes.io/blog/2018/10/09/introducing-volume-snapshot-alpha-for-kubernetes/).
* `snapshotter` field to point to the respective CSI driver that is responsible for taking snapshot. As we are using [GCE Persistent Disk CSI Driver](https://github.com/kubernetes-sigs/gcp-compute-persistent-disk-csi-driver), we are going to use `pd.csi.storage.gke.io` in this field.

Let's create the `volumeSnapshotClass` crd we have shown above,
```console
$ kubectl apply -f ./docs/examples/volume-snapshot/default-volumesnapshotclass.yaml
volumesnapshotclass.snapshot.storage.k8s.io/default-snapshot-class created
```

To keep everything isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$ kubectl create ns demo
namespace/demo created
```

>Note: YAML files used in this tutorial are stored in [/docs/examples/volume-snapshot](/docs/examples/volume-snapshot) directory of [appscode/stash](https://github.com/stashed/stash) repository.

### Take Volume Snapshot

When you create a Statefulset, there is no need to create PVCs separately, because all replicas in Statefulset use different PVCs to store data. Kubenetes allows us to define a `volumeClaimTemplates` in Statefulset so that new PVC is created for each replica automatically. 
We will take snapshot of those PVCs using Stash.

#### Deploy Statefulset

Now, We will deploy a Statefulset. This Statefulset will automatically generate sample data (sample-file.txt file) in `/source/data` directory.

Sample Steateful YAML are given below,

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
        app: stash # Pod template's label selector
    spec:
      containers:
        - args: ["touch source/data/sample-file.txt && sleep 3000"]
          command: ["/bin/sh", "-c"]
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
        name: source-pvc
        namespace: demo
      spec:
        accessModes:
          - ReadWriteOnce
        storageClassName: standard
        resources:
          requests:
            storage: 6Gi
```
Let's create the Statefulset we have shown above.

```console
$ kubectl create -f ./docs/examples/volume-snapshot/statefulset/statefulset.yaml 
service/svc created
statefulset.apps/stash-demo created
```

Now, wait for Statefulset’s pod to go into the Running state.

```console
$ $ kubectl get pod -n demo
NAME           READY   STATUS    RESTARTS   AGE
stash-demo-0   1/1     Running   0          97s
stash-demo-1   1/1     Running   0          67s
stash-demo-2   1/1     Running   0          39s
```
You can see that automatically created PVCs are bound to the corresponding pod,
```yaml
kubectl get pvc -n demo
NAME                          STATUS   VOLUME                                          CAPACITY   ACCESS MODES   STORAGECLASS   AGE
source-pvc-stash-demo-0       Bound    pvc-6456fb74-8382-11e9-91dc-42010a80001a        6Gi        RWO            standard       27m
source-pvc-stash-demo-1       Bound    pvc-76564e29-8382-11e9-91dc-42010a80001a        6Gi        RWO            standard       26m
source-pvc-stash-demo-2       Bound    pvc-8730dbb2-8382-11e9-91dc-42010a80001a        6Gi        RWO            standard       26m
```
Verify that the sample data has been created in `/source/data` directory for `stash-demo-0` , `stash-demo-1` and `stash-demo-2` pod respectively using the following command,

```console
$ kubectl exec -n demo stash-demo-0 -- ls -R /source/data
/source/data:
lost+found
sample-file.txt
$ kubectl exec -n demo stash-demo-1 -- ls -R /source/data
/source/data:
lost+found
sample-file.txt
$ kubectl exec -n demo stash-demo-2 -- ls -R /source/data
/source/data:
lost+found
sample-file.txt
```

#### Create BackupConfiguration

Now, create a `BackupConfiguration` crd to take snapshot of the PVCs of `stash-demo` Statefulset.
Sample `BackupConfiguration` crd YAML are given below,

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
* `spec.schedule` is a [cron expression](https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/#schedule) indicates that `BackupSession` will be created at 1 minute interval.

* `spec.driver` indicates the name of the agent to use to back up the target. Currently, Stash supports `Restic`, `VolumeSnapshotter` drivers. The `VolumeSnapshotter` is used to backup/restore PVC using `VolumeSnapshot` API.

* `spec.replicas` specifies the number of replicas whose data should be backed up. i.e. In statefulset for value 1, it will take snapshot of  `<claim-name>-<statefulset-name>-0` volume; for replica 2, it will take snapshot of `<claim-name>-<statefulset-name>-0` and `<claim-name>-<statefulset-name>-1` volume and so on.

* `spec.target.ref`  refers to the backup target. `apiVersion`, `kind` and `name` refers to the `apiVersion`, `kind` and `name` of the targeted workload respectively. Stash will use this information to create a Volume Snapshotter Job for creating VolumeSnapshot.

* `spec.target.snapshotClassName` indicates the [VolumeSnapshotClass](https://kubernetes.io/docs/concepts/storage/volume-snapshot-classes/) to use for volume snapshotting.



#### Create Snapshot for one Replica

Now, create a `BackupConfiguration` crd to take a snapshot of `stash-demo-0` volume. For that, we will set the `spec.replica` to 1 in `BackupConfiguration` crd.

Let's create the `BackupConfiguration` crd we have shown above.

```console
$ kubectl apply -f ./docs/examples/volume-snapshot/statefulset/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/statefulset-volume-snapshot created
```
##### Verify CronJob:

If everything goes well, Stash will create a `CronJob` to take periodic snapshot of `stash-demo-0` volume of the Statefulset with the schedule specified in `spec.schedule` field of `BackupConfiguration` crd.

 Check that the `CronJob` has been created using the following command,

```console
$ kubectl get cronjob -n demo
NAME                          SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
statefulset-volume-snapshot   */1 * * * *   False     0        <none>          18s
```

##### Wait for BackupSession:

The `statefulset-volume-snapshot` CronJob will trigger a backup on each schedule by creating a `BackpSession` crd.
 
Wait for a schedule to appear. Run the following command to watch `BackupSession` crd,

```console
$ watch -n 1 kubectl get backupsession -n demo
Every 1.0s: kubectl get backupsession -n demo                      suaas-appscode: Tue Jun 18 18:35:41 2019

NAME                                     BACKUPCONFIGURATION           PHASE        AGE
statefulset-volume-snapshot-1559297711   statefulset-volume-snapshot   Running      57s
statefulset-volume-snapshot-1559297711   statefulset-volume-snapshot   Succeeded    57s
```

We can see above that the backup session has succeeded. Now, we will verify that the VolumeSnapshot has been created and the snapshots has been stored in the respective backend.

##### Verify Volume Snapshotting and Backup:

Once a `BackupSession` crd is created, it creates volume snapshotter `Job`. Then the `Job` creates `VolumeSnapshot` crd for the targeted PVC.The `VolumeSnapshot` name follows the following pattern:

```
 <PVC name>-<backup session creation timestamp in Unix epoch seconds>
```

Check that the `VolumeSnapshot` has been created Successfully.

```console
$ kubectl get volumesnapshot -n demo 
NAMESPACE   NAME                                  AGE
demo        source-pvc-stash-demo-0-1559297711    105s
```

You will also see that, The YAML for ` source-pvc-stash-demo-0-1559297711` looks like using the following command,

```console
$ kubectl get volumesnapshot source-pvc-stash-demo-0-1559297711 -n demo -o yaml
apiVersion: snapshot.storage.k8s.io/v1alpha1
kind: VolumeSnapshot
metadata:
  creationTimestamp: "2019-06-17T12:15:12Z"
  finalizers:
  - snapshot.storage.kubernetes.io/volumesnapshot-protection
  generation: 4
  name: source-pvc-stash-demo-0-1559297711
  namespace: demo
  resourceVersion: "919278"
  selfLink: /apis/snapshot.storage.k8s.io/v1alpha1/namespaces/demo/volumesnapshots/source-pvc-stash-demo-0-1559297711
  uid: 8e7d2e6d-90f9-11e9-bd3e-42010a800011
spec:
  snapshotClassName: default-snapshot-class
  snapshotContentName: snapcontent-8e7d2e6d-90f9-11e9-bd3e-42010a800011
  source:
    apiGroup: null
    kind: PersistentVolumeClaim
    name: source-data-stash-demo-0
status:
  creationTime: "2019-06-17T12:15:13Z"
  readyToUse: true
  restoreSize: 6Gi
```

If we navigate to the `Snapshots` in the GCS navigation menu, we will see snapshots has been stored successfully.

<p align="center">
  <img alt="Stash Backup Flow" src="/docs/images/v1beta1/backends/volumesnapshot/statefulset2.png">
<figcaption align="center">Fig: Snapshot in GCE Bucket</figcaption>
</p>

#### Create Snapshot for three Replica

Again, create a `BackupConfiguration` crd to take snapshot of `stash-demo-0` , `stash-demo-1` and `stash-demo-2` volume simultaneously. For that we will set the `spec.replica` to 3 in `BackupConfiguration` crd. 

Let's create the `BackupConfiguration` crd we have shown above.

```console
$ kubectl apply -f ./docs/examples/volume-snapshot/statefulset/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/statefulset-volume-snapshot created
```

##### Verify CronJob:

If everything goes well, Stash will create a `CronJob` to take periodic snapshot of `stash-demo-0` , `stash-demo-1` and `stash-demo-2` volumes of the Statefulset with the schedule specified in `spec.schedule` field of `BackupConfiguration` crd.

 Check that the `CronJob` has been created using the following command,

```console
$ kubectl get cronjob -n demo
NAME                          SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
statefulset-volume-snapshot   */1 * * * *   False     0        <none>          18s
```

##### Wait for BackupSession:

The `statefulset-volume-snapshot` CronJob will trigger a backup on each schedule by creating a `BackpSession` crd.
 
Wait for a schedule to appear. Run the following command to watch `BackupSession` crd,

```console
$ watch -n 1 kubectl get backupsession -n demo
Every 1.0s: kubectl get backupsession -n demo                      suaas-appscode: Tue Jun 18 18:35:41 2019

NAME                                     BACKUPCONFIGURATION           PHASE        AGE
statefulset-volume-snapshot-1560231666   statefulset-volume-snapshot   Running      57s
statefulset-volume-snapshot-1560231666   statefulset-volume-snapshot   Succeeded    57s
```

We can see above that the backup session has succeeded. Now, we will verify that the VolumeSnapshot has been created and the snapshots has been stored in the respective backend.

##### Verify Volume Snapshotting and Backup:

Once a `BackupSession` crd is created, it creates volume snapshotter `Job`. Then the `Job` creates `VolumeSnapshot` crd for the targeted PVC.The `VolumeSnapshot` name follows the following pattern:

```
 <PVC name>-<statefulset name>-<pod ordinal>-<backup session creation timestamp in Unix epoch seconds>
```

Check that the `VolumeSnapshot` has been created Successfully.

```console
$ kubectl get volumesnapshot -n demo
NAME                                  AGE
source-pvc-stash-demo-0-1560231666    105s
source-pvc-stash-demo-1-1560231666    105s
source-pvc-stash-demo-2-1560231666    105s
```

You will also see that, The YAML for ` source-pvc-stash-demo-0-1560231666` looks like using the following command,

```console
$ kubectl get volumesnapshot source-pvc-stash-demo-0-1560231666 -n demo -o yaml
apiVersion: snapshot.storage.k8s.io/v1alpha1
kind: VolumeSnapshot
metadata:
  creationTimestamp: "2019-06-17T12:15:12Z"
  finalizers:
  - snapshot.storage.kubernetes.io/volumesnapshot-protection
  generation: 4
  name: source-pvc-stash-demo-0-1560773710
  namespace: demo
  resourceVersion: "919278"
  selfLink: /apis/snapshot.storage.k8s.io/v1alpha1/namespaces/demo/volumesnapshots/source-pvc-stash-demo-0-1560231666
  uid: 912b1ad2-90f9-11e9-bd3e-42010a800011
spec:
  snapshotClassName: default-snapshot-class
  snapshotContentName: snapcontent-912b1ad2-90f9-11e9-bd3e-42010a800011
  source:
    apiGroup: null
    kind: PersistentVolumeClaim
    name: source-data-stash-demo-0
status:
  creationTime: "2019-06-17T12:15:13Z"
  readyToUse: true
  restoreSize: 6Gi
```

If we navigate to the `Snapshots` in the GCS navigation menu, we will see snapshots has been stored successfully.

<p align="center">
  <img alt="Stash Backup Flow" src="/docs/images/v1beta1/backends/volumesnapshot/statefulset.png">
<figcaption align="center">Fig: Snapshots in GCE Bucket</figcaption>
</p>

### Restore PVC from VolumeSnapshot

This section will show you how to restore the PVCs from the snapshots we have taken in earlier section.

#### Create RestoreSession

At first, we have to create a `RestoreSession` crd to restore the PVCs from respective snapshot.
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
          name: source-data-stash-demo
        spec:
          accessModes: [ "ReadWriteOnce" ]
          storageClassName: "standard"
          resources:
            requests:
              storage: 6Gi
          dataSource:
            kind: VolumeSnapshot
            name: ${CLAIM_NAME}-${POD_ORDINAL}-1560231666
            #name: ${CLAIM_NAME}-0-1560231666
            apiGroup: snapshot.storage.k8s.io
```
Here,

* `spec.target.volumeClaimTemplates`:
  * `metadata.name` is the name of the restored `PVC` or prefix of the `VolumeSnapshot` name.
  * `spec.dataSource`: `spec.dataSource` specifies the source of the data from where the newly created PVC will be intialized. It requires following fields to set:
    * `apiGroup` is the group for resource being referenced. Now, kubernetes supports only `snapshot.storage.k8s.io`.
    * `kind` is resource of the kind being referenced. Now, kubernetes supports only `VolumeSnapshot`.
    * `name` is the `VolumeSnapshot` resource name. In `RestoreSession` crd, You can template the name by using the following convention:
    ```
     ${CLAIM_NAME}-${POD_ORDINAL}-<backup session creation timestamp in Unix epoch seconds>
     ```  
     or 
     ```
     ${CLAIM_NAME}-<backup session creation timestamp in Unix epoch seconds>
     ```
    `${CLAIM_NAME}` and `{POD_ORDINAL}` are resolved by the Stash operator and replaced by the `metadata.name` and pod ordinal respectively. You can set the snapshot name directly.

#### Restore from different VolumeSnapshot

Let's create the `RestoreSession` crd we have shown above.

```console
$ kubectl create -f ./docs/examples/volume-snapshot/statefulset/restoresession.yaml
restoresession.stash.appscode.com/restore-pvc created
````
Once, you have created the `RestoreSession` crd, Stash will create a job to restore. We can watch the `RestoreSession` phase to check if the restore process is succeeded or not.

Run the following command to watch RestoreSession phase,

```console
$ watch -n 1 kubectl get restore -n demo
Every 1.0s: kubectl get restore -n demo                      suaas-appscode: Tue Jun 18 18:35:41 2019

NAME          REPOSITORY-NAME   PHASE       AGE
restore-pvc                     Running     10s
restore-pvc                     Succeeded   1m
```
So, we can see from the output of the above command that the restore process succeeded.

##### Verify Restored PVC:

Once a restore process is complete, we will see that new PVCs with the name `source-pvc-stash-demo-0` , `source-pvc-stash-demo-1` and `source-pvc-stash-demo-2` has been created successfully.

check that the status of the PVC is bound,

```console
$  kubectl get pvc -n demo
NAME                      STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
source-pvc-stash-demo-0   Bound    pvc-7c773921-8c10-11e9-bd3e-42010a800011   6Gi        RWO            standard       10m
source-pvc-stash-demo-1   Bound    pvc-7c7b32ad-8c10-11e9-bd3e-42010a800011   6Gi        RWO            standard       10m
source-pvc-stash-demo-2   Bound    pvc-7c7d0bdf-8c10-11e9-bd3e-42010a800011   6Gi        RWO            standard       10m
````

##### Verify Restored Data:

We will create a new Statefulset to verify whether the restored data has been restored successfully. 

Below, the YAML for the Statefulset we are going to create.

```console
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
        app: stash # Pod template's label selector
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
        - name: source-data-0
          mountPath: /source/data
        - name: source-data-1
          mountPath: /source/data
        - name: source-data-2
          mountPath: /source/data
      volumes:
      - name: source-data-0
        persistentVolumeClaim:
          claimName: source-pvc-stash-demo-0
      - name: source-data-1
        persistentVolumeClaim:
          claimName: source-pvc-stash-demo-1
      - name: source-data-2
        persistentVolumeClaim:
          claimName: source-pvc-stash-demo-2
```

Let's create the Statefulset we have shown above.

```console
$ kubectl create -f ./docs/examples/volume-snapshot/statefulset/statefulset.yaml 
service/svc created
statefulset.apps/stash-demo created
```

Now, wait for Statefulset’s pod to go into the Running state.

```console
$ $ kubectl get pod -n demo
NAME           READY   STATUS    RESTARTS   AGE
stash-demo-0   1/1     Running   0          97s
stash-demo-1   1/1     Running   0          67s
stash-demo-2   1/1     Running   0          39s
```
You can see that automatically created PVCs are bound to the corresponding pod,
```yaml
kubectl get pvc -n demo
NAME                          STATUS   VOLUME                                          CAPACITY   ACCESS MODES   STORAGECLASS   AGE
source-pvc-stash-demo-0       Bound    pvc-6456fb74-8382-11e9-91dc-42010a80001a        6Gi        RWO            standard       27m
source-pvc-stash-demo-1       Bound    pvc-76564e29-8382-11e9-91dc-42010a80001a        6Gi        RWO            standard       26m
source-pvc-stash-demo-2       Bound    pvc-8730dbb2-8382-11e9-91dc-42010a80001a        6Gi        RWO            standard       26m
```
Verify that the sample data has been created in `/source/data` directory for `stash-demo-0` , `stash-demo-1` and `stash-demo-2` pod respectively using the following command,

```console
$ kubectl exec -n demo stash-demo-0 -- ls -R /source/data
/source/data:
lost+found
sample-file.txt
$ kubectl exec -n demo stash-demo-1 -- ls -R /source/data
/source/data:
lost+found
sample-file.txt
$ kubectl exec -n demo stash-demo-2 -- ls -R /source/data
/source/data:
lost+found
sample-file.txt
```

#### Restore from same VolumeSnapshot

Again at first, set the `.spec.dataSource.name` to `{ClAIM_NAME}-0-1560231666` in `RestoreSession` and create the `RestoreSession` crd we have shown above.

```console
$ kubectl create -f ./docs/examples/volume-snapshot/statefulset/restoresession.yaml
restoresession.stash.appscode.com/restore-pvc created
````
Once, you have created the `RestoreSession` crd, Stash will create a job to restore. We can watch the `RestoreSession` phase to check if the restore process is succeeded or not.

Run the following command to watch RestoreSession phase,

```console
$ watch -n 1 kubectl get restore -n demo
Every 1.0s: kubectl get restore -n demo                      suaas-appscode: Tue Jun 18 18:35:41 2019

NAME          REPOSITORY-NAME   PHASE       AGE
restore-pvc                     Running     10s
restore-pvc                     Succeeded   1m
```
So, we can see from the output of the above command that the restore process succeeded.

##### Verify Restored PVC:

Once a restore process is complete, we will see that new PVCs with the name `source-pvc-stash-demo-0` , `source-pvc-stash-demo-1` and `source-pvc-stash-demo-2` has been created successfully.

check that the status of the PVC is bound,

```console
$  kubectl get pvc -n demo
NAME                       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
source-data-stash-demo-0   Bound    pvc-7c773921-8c10-11e9-bd3e-42010a800011   6Gi        RWO            standard       10m
source-data-stash-demo-1   Bound    pvc-7c7b32ad-8c10-11e9-bd3e-42010a800011   6Gi        RWO            standard       10m
source-data-stash-demo-2   Bound    pvc-7c7d0bdf-8c10-11e9-bd3e-42010a800011   6Gi        RWO            standard       10m
````

##### Verify Restored Data:

We will create a new Statefulset to verify whether the restored data has been restored successfully. 

Below, the YAML for the Statefulset we are going to create.

```console
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
        app: stash # Pod template's label selector
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
        - name: source-data-0
          mountPath: /source/data
        - name: source-data-1
          mountPath: /source/data
        - name: source-data-2
          mountPath: /source/data
      volumes:
      - name: source-data-0
        persistentVolumeClaim:
          claimName: source-pvc-stash-demo-0
      - name: source-data-1
        persistentVolumeClaim:
          claimName: source-pvc-stash-demo-1
      - name: source-data-2
        persistentVolumeClaim:
          claimName: source-pvc-stash-demo-2
```

Let's create the Statefulset we have shown above.

```console
$ kubectl create -f ./docs/examples/volume-snapshot/statefulset/statefulset.yaml 
service/svc created
statefulset.apps/stash-demo created
```

Now, wait for Statefulset’s pod to go into the Running state.

```console
$ $ kubectl get pod -n demo
NAME           READY   STATUS    RESTARTS   AGE
stash-demo-0   1/1     Running   0          97s
stash-demo-1   1/1     Running   0          67s
stash-demo-2   1/1     Running   0          39s
```
You can see that automatically created PVCs are bound to the corresponding pod,
```yaml
kubectl get pvc -n demo
NAME                          STATUS   VOLUME                                          CAPACITY   ACCESS MODES   STORAGECLASS   AGE
source-pvc-stash-demo-0       Bound    pvc-6456fb74-8382-11e9-91dc-42010a80001a        6Gi        RWO            standard       27m
source-pvc-stash-demo-1       Bound    pvc-76564e29-8382-11e9-91dc-42010a80001a        6Gi        RWO            standard       26m
source-pvc-stash-demo-2       Bound    pvc-8730dbb2-8382-11e9-91dc-42010a80001a        6Gi        RWO            standard       26m
```
Verify that the sample data has been created in `/source/data` directory for `stash-demo-0` , `stash-demo-1` and `stash-demo-2` pod respectively using the following command,

```console
$ kubectl exec -n demo stash-demo-0 -- ls -R /source/data
/source/data:
lost+found
sample-file.txt
$ kubectl exec -n demo stash-demo-1 -- ls -R /source/data
/source/data:
lost+found
sample-file.txt
$ kubectl exec -n demo stash-demo-2 -- ls -R /source/data
/source/data:
lost+found
sample-file.txt
```

### Cleaning Up

To clean up the Kubernetes resources created by this tutorial, run:

```console
$ kubectl delete -n demo statefulset stash-demo
$ kubectl delete -n demo backupconfiguration statefulset-volume-snapshot
$ kubectl delete -n demo restoresession restore-pvc
$ kubectl delete -n demo storageclass standard
$ kubectl delete -n demo volumesnapshotclass default-snapshot-class
```