## Snapshot Statefulste's Volumes

This guide will show you how to use Stash to snapshot Statefulset's volumes and restore PersistentVolumeClaims from snapshot using Kubernetes [VolumeSnapshot](https://kubernetes.io/docs/concepts/storage/volume-snapshots/) API.

### Before You Begin

* At first, you need to familiarize with the [GCE Persistent Disk CSI Driver](https://github.com/kubernetes-sigs/gcp-compute-persistent-disk-csi-driver)
* Also need to enable the Kubernetes VolumeSnapshotDataSource alpha feature via feature gate 
* The kubectl command-line tool must be configured to communicate with your cluster.
* And need to install Stah Operator in your cluster. 
If you are not familiar with that please visit [here](docs/guides/latest/volumesnapshot/backup.md) 

#### Prepare for VolumeSnapshot

If you don't already have a StorageClass that uses the CSI driver that supports VolumeSnapshot feature, create one first. Here, we are going to create StorageClass that uses GCE Persistent Disk CSI Driver.
Smaple `StorageClass` YAML are given below,

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
The [volumeBindingMode](https://kubernetes.io/docs/concepts/storage/storage-classes/#volume-binding-mode) field controls when volume binding and dynamic provisioning should occur. Kubernetes specifies `Immediate` and `WaitForFirstConsumer` volumeBindingMode. The `Immediate` mode indicates that volume binding and dynamic provisioning occurs once the PVC is created and `WaitForFirstConsumer` mode indicates that volume binding and provisioning does not occur until a pod using the PVC is created.

Let's create the StorageClass we have shown above.

```console
$ kubectl apply -f ./docs/examples/volume-snapshot/storageclass.yaml
storageclass.storage.k8s.io/standard created
```
Also create a `VolumeSnapshotClass` for further uses.

Sample `VolumeSnapshotClass` crd YAML are given below,

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

* `metadata.annotations` annotations are used to set default [volumeSnapshotClass](https://kubernetes.io/blog/2018/10/09/introducing-volume-snapshot-alpha-for-kubernetes/)
* `snapshotter` field to point to determined CSI driver, for GCE CSI driver, it will be `pd.csi.storage.gke.io`.

Let's create the `volumeSnapshotClass` crd we have shown above.

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

When you create a Statefulset, there is no need to create PVCs seperately, because all replicas in Statefulset use different PVCs to store data. Kubenetes allows us to define a `volumeClaimTemplates` so that a new PVC is created for each replica automatically.
Now we are going to create snapshot for the number of replica 1.

#### Deploy Workload:

At first, you need to create a headless Service to control Statefulset network domain.
Sample Service YAML are given below,

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
```
Let's create the Service we have shown above,

```console
$ kubectl apply -f ./docs/examples/volume-snapshot/statefulset/svc.yaml
service/svc created
```
Now, deploy a Statefulset that defines `volumeClaimTemplates`.
Sample Steateful YAML are given below,

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: stash-demo
  namespace: demo
spec:
  selector:
    matchLabels:
      app: demo
  serviceName: svc
  replicas: 1
  template:
    metadata:
      labels:
        app: demo # Pod template's label selector
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
      name: source-data
      namespace: demo
    spec:
      accessModes: 
        - ReadWriteOnce
      storageClassName: "standard"
      resources:
        requests:
          storage: 6Gi
```
Let's create the Statefulset we have shown above.

```console
$ kubectl create -f ./statefulset/statefulset.yaml 
statefulset.apps/stash-demo created
```

Now, wait for Statefulset’s pod to go into the Running state.

```console
$ kubectl get pod -n demo
NAME           READY   STATUS    RESTARTS   AGE
stash-demo-0   1/1     Running   0          23s
```

You can check that the `/source/data/` directory of this pod is populated with data from the `backup-pvc` PVC by using the following command,

```console
$ kubectl exec -n demo stash-demo-0 -- ls -R /source/data
/source/data:
lost+found
sample-file.txt
```

#### Create BackupConfiguration:

Now, create a `BackupConfiguration` crd to take snapshot of `backup-pvc` PVC. Once the  `BackupConfiguration` crd is created, Stash will create a CronJob to take periodic snapshot of `backp-pvc` volume.

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
* `spec.schedule` is a [cron expression](https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/#schedule) indicates that `BackupSession` will be created every 1 minute.

* `spec.driver` indicates the name of the agent to use to back up the target. Currently, Stash supports Restic, VolumeSnapshotter drivers. The VolumeSnapshotter is used to backup/restore PVC using VolumeSnapshot API.

* `spec.replicas` are the desired number of replicas whose data should be backed up.

* `spec.target.ref`  refers to the backup target. `apiVersion`, `kind` and `name` refers to the `apiVersion`, `kind` and `name` of the targeted workload respectively. Stash will use this information to create a Volume Snapshotter Job for Creating VolumeSnapshot.

* `spec.target.snapshotClassName` indicates the [VolumeSnapshotClass](https://kubernetes.io/docs/concepts/storage/volume-snapshot-classes/) to use for volume snapshotting.

Let's create the `BackupConfiguration` crd we have shown above.

```console
$ kubectl apply -f ./docs/examples/volume-snapshot/statefulset/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/statefulset-volume-snapshot created
```
Here, `statefulset-volume-snapshot` is the name of the `BackupConfiguration` crd that has been created successfully.

#### Verify Volume Snapshotting 

If everything goes well, Stash operator creates CronJob which creates BackupSession crd on each schedule. `BackupSession` crd name will be automatically generated by the following pattern `<BackupConfiguration name>-<BackupSession creation timestamp in Unix epoch seconds>` and label should be set as the following pattern `stash.appscode.com/backup-configuration:<BackupConfiguration name>`.

Now, wait a few minutes,  you will see that `BackupSession` has been created.
Check that the phase of `BackupSession` status is `Running` 

```console
kubectl get bs -n demo -l   stash.appscode.com/backup-configuration=statefulset-volume-snapshot
NAME                                     BACKUPCONFIGURATION           PHASE       AGE
statefulset-volume-snapshot-1559281449   statefulset-volume-snapshot   Running     40s
```
Here, `statefulset-volume-snapshot-1559281449` is the name of the `BackupSession` crd that has been created successfully after 1 minute.

Stash operator watches for BackupSession crd and it will create a volume snapshotter job named `volume-snapshot-<BackupSession name>`.
Check it by using the following command

```console
$ kubectl get job -n demo volume-snapshot-statefulset-volume-snapshot-1559281449
NAMESPACE   NAME                                                     COMPLETIONS   DURATION   AGE
demo        volume-snapshot-statefulset-volume-snapshot-1559281449   0/1           19s        20s
```
Finally, volume snapshotter job will create `VolumeSnapshot` crd for the targeted PVCs. The `VolumeSnapshot` names will follow the following pattern `<PVC name>-<statefulset name>-<pod ordinal>-<backup session creation timestamp in Unix epoch seconds>`.

Wait a few minutes,  you will see that `source-data-stash-demo-0-1559281449` VolumeSnapshot has been created Successfully.

```console
$ kubectl get volumesnapshot -n demo source-data-stash-demo-0-1559281449
NAMESPACE   NAME                                         AGE
demo        source-data-stash-demo-0-1559281449          153m
```
Once the snapshotting is completed, BackupSession crd phase will be Succeeded. Check the BackupSession phase is Succeeded by following command: 

```console
$ kubectl get bs -n demo -l   stash.appscode.com/backup-configuration=statefulset-volume-snapshot
NAME                                     BACKUPCONFIGURATION           PHASE       AGE
statefulset-volume-snapshot-1559281449   statefulset-volume-snapshot   Succeeded   71s
```
### Restore PVC from VolumeSnapshot

At first, you need to have VolumeSnapshot taken by Stash. If you already don’t have any VolumeSnapshot, create one from [here](docs/guides/latest/volumesnapshot/deployment-backup.md).

#### Create RestoreSession

Now, create a `RestoreSession` crd to restore PersistentVolumeClaims from VolumeSnapshot.
Sample `RestoreSession` crd YAML are given below,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: restore-pvc
  namespace: demo
spec:
  driver: VolumeSnapshotter
  target:
    replicas : 1
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
            name: ${CLAIM_NAME}-${POD_ORDINAL}-1559281449
            apiGroup: snapshot.storage.k8s.io
```
Here,

* `spec.driver` indicates the name of the agent to use to back up the target. Currently, Stash supports Restic, VolumeSnapshotter drivers. The VolumeSnapshotter is used to backup/restore PVC using VolumeSnapshot API.

* `spec.target.replicas` specifies the number of source to restore from.

* `spec.target.volumeClaimTemplates[*].metadata.name` is the name of the VolumeSnapshot without pod ordinal and timestamp portion, i.e. for a VolumeSnapshot name `source-data-stash-demo-0-1559281449`, this field will be `source-data-stash-demo`.

* `spec.target.volumeClaimTemplates[*].spec.storageClassName` indicates the name of the [storageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/) required by the claim.

* `spec.target.volumeClaimTemplates.spec.dataSource` are used only for `VolumeSnapshot`. currently `VolumeSnapshot` is the only supported data source. It will create a new volume and data will be restored to the volume at the same time.
  * `kind` is the type of resource being referenced.

  * `name` is the name of the resource being referenced. In `RestoreSession` crd, The name follows the following convention `${CLAIM_NAME}-${POD_ORDINAL}-<backup session creation timestamp in Unix epoch seconds>`. `${CLAIM_NAME}` and `{POD_ORDINAL}` are resolved by the Stash operator and replaced by the PVCs name and pod ordinal of the Statesulset replicas.

Let's create the `RestoreSession` crd we have shown above.

```console
$ kubectl create -f ./docs/examples/volume-snapshot/statefulset/restoresession.yaml
restoresession.stash.appscode.com/restore-pvc created
````
Here, restore-pvc is the name of the RestoreSession that has been created.

#### Verify Restored PVC 

If everything goes well, Stash operator creates a Restore Job. Then Restore Job will create new PVC.  

Now, wait a few minutes, you will see that a new PVC with the name `source-data-stash-demo-0` has been created successfully.

check that the status of the PVC is bound

```console
$  kubectl get pvc -n demo source-data-stash-demo-0
NAME                       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
source-data-stash-demo-0   Bound    pvc-b6b32265-836b-11e9-91dc-42010a80001a   6Gi        RWO            standard       2m
````



### Cleaning Up

To clean up the Kubernetes resources created by this tutorial, run:

```console
$ kubectl delete -n demo statefulset stash-demo
$ kubectl delete -n demo backupconfiguration statefulset-volume-snapshot
$ kubectl delete -n demo restoresession restore-pvc
$ kubectl delete -n demo storageclass standard
$ kubectl delete -n demo volumesnapshotclass default-snapshot-class
```


