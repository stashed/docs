## Snapshot Standalone PVC's

This guide will show you how to use Stash to snapshot standalone PersistentVolumeClaims and restore PersistentVolumeClaims from snapshot using Kubernetes [VolumeSnapshot](https://kubernetes.io/docs/concepts/storage/volume-snapshots/) API.

### Before You Begin

* At first, you need to familiarize with the [GCE Persistent Disk CSI Driver](https://github.com/kubernetes-sigs/gcp-compute-persistent-disk-csi-driver)
* Also need to enable the Kubernetes VolumeSnapshotDataSource alpha feature via feature gate 
* The kubectl command-line tool must be configured to communicate with your cluster.
* And need to install Stah Operator in your cluster. 
If you are not familiar with that please visit [here](docs/guides/latest/volumesnapshot/backup.md) 

#### Prepare for VolumeSnapshot

If you don't already have a StorageClass that uses the CSI driver that supports VolumeSnapshot feature, create one first. Here, we are going to create StorageClass that uses GCE Persistent Disk CSI Driver.
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
volumeBindingMode: WaitForFirstConsumer
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

At first, You will create a PVC.
Sample PVC YAML are given below,

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
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
Let's create the PVC we have shown above.

```console
$ kubectl apply -f ./docs/examples/volume-snapshot/standalonePVC/source-pvc.yaml
persistentvolumeclaim/source-pvc created
```
#### Create Pod:

Now, create a Pod that uses the above PVC and create some sample data to store in PVC.
Below, the YAML for the Pod we are going to create.

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
      args: ["touch /demo/data/sample-file.txt && sleep 3000"]
      volumeMounts:
        - name: source-data
          mountPath: /demo/data
  volumes:
    - name: source-data
      persistentVolumeClaim:
        claimName: source-pvc
        readOnly: false
```
Let's create the Pod we have shown above.

```console
$ kubectl apply -f ./docs/examples/volume-snapshot/standalonePVC/source-pod.yaml
pod/source-pod created
```

Now, wait for Pod to go into the Running state.

```console
$ kubectl get pod -n demo 
NAME         READY   STATUS    RESTARTS   AGE
source-pod   1/1     Running   0          25s
```

You can check that the `/demo/data/` directory of this pod is populated with data by using the following command,

```console
$ kubectl exec -n demo source-pod  -- ls -R /demo/data
/demo/data:
lost+found
sample-file.txt
```

#### Create BackupConfiguration:

Now, create a `BackupConfiguration` crd to take a snapshot of `source-pvc` PVC. Once the  `BackupConfiguration` crd is created, Stash will create a CronJob to take a periodic snapshot of `source-pvc` volume.

Sample `BackupConfiguration` crd YAML are given below,

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
      name: source-pvc
    snapshotClassName: default-snapshot-class
  retentionPolicy:
    name: 'keep-last-5'
    keepLast: 5
    prune: true
```

Here,
* `spec.schedule` is a [cron expression](https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/#schedule) indicates that `BackupSession` will be created every 1 minute.

* `spec.driver` indicates the name of the agent to use to back up the target. Currently, Stash supports Restic, VolumeSnapshotter drivers. The VolumeSnapshotter is used to backup/restore PVC using VolumeSnapshot API.

* `spec.target.ref`  refers to the backup target. `apiVersion`, `kind` and `name` refers to the `apiVersion`, `kind` and `name` of the targeted workload respectively. Stash will use this information to create a Volume Snapshotter Job for Creating VolumeSnapshot.

* `spec.target.snapshotClassName` indicates the [VolumeSnapshotClass](https://kubernetes.io/docs/concepts/storage/volume-snapshot-classes/) to use for volume snapshotting.

Let's create the `BackupConfiguration` crd we have shown above.

```console
$ kubectl apply -f ./docs/examples/volume-snapshot/standalonePVC/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/pvc-volume-snapshot created
```
Here, `pvc-volume-snapshot` is the name of the `BackupConfiguration` crd that has been created successfully.

#### Verify Volume Snapshotting 

If everything goes well, Stash operator creates CronJob which creates BackupSession crd on each schedule. `BackupSession` crd name will be automatically generated by the following pattern `<BackupConfiguration name>-<BackupSession creation timestamp in Unix epoch seconds>` and label should be set as the following pattern `backup-configuration:<BackupConfiguration name>`.

Now, wait a few minutes,  you will see that `BackupSession` has been created.
Check that the phase of `BackupSession` status is `Running` 

```console
$ 
kubectl get bs -n demo -l  stash.appscode.com/backup-configuration=pvc-volume-snapshot
NAME                             BACKUPCONFIGURATION   PHASE     AGE
pvc-volume-snapshot-1560400745   pvc-volume-snapshot   Running   5s
```
Here, `pvc-volume-snapshot-1560400745` is the name of the `BackupSession` crd that has been created successfully after 1 minute.

Stash operator watches for `BackupSession` crd and it will create a volume snapshotter job named `volume-snapshot-<BackupSession name>`.
Check it by using the following command

```console
$ kubectl get job -n demo volume-snapshot-pvc-volume-snapshot-1560400745
NAMESPACE   NAME                                                     COMPLETIONS   DURATION   AGE
demo        volume-snapshot-pvc-volume-snapshot-1560400745           1/1           19s        40s
```
Finally, volume snapshotter job will create `VolumeSnapshot` crd for the targeted PVCs. The `VolumeSnapshot` names will follow the following pattern `<PVC name>-<backup session creation timestamp in Unix epoch seconds>`.

Wait a few minutes,  you will see that `source-pvc-1560400745` VolumeSnapshot has been created Successfully.

```console
$ 
kubectl get volumesnapshot -n demo source-pvc-1560400745
NAME                    AGE
source-pvc-1560400745   1m30s
```
Once the snapshotting is completed, the BackupSession crd phase will be Succeeded. Check the BackupSession phase is Succeeded by the following command: 

```console
$ kubectl get bs -n demo -l  backup-configuration=pvc-volume-snapshot
NAME                                     BACKUPCONFIGURATION           PHASE       AGE
pvc-volume-snapshot-1560400745           pvc-volume-snapshot           Succeeded   1m32s
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
    volumeClaimTemplates:
      - metadata:
          name: source-pvc
        spec:
          accessModes: [ "ReadWriteOnce" ]
          storageClassName: "standard"
          resources:
            requests:
              storage: 6Gi
          dataSource:
            kind: VolumeSnapshot
            name: ${CLAIM_NAME}-1560400745
            apiGroup: snapshot.storage.k8s.io
```
Here,

* `spec.driver` indicates the name of the agent to use to back up the target. Currently, Stash supports Restic, VolumeSnapshotter drivers. The VolumeSnapshotter is used to backup/restore PVC using VolumeSnapshot API.

* `spec.target.replicas` specifies the number of the source to restore from.

* `spec.target.volumeClaimTemplates[*].metadata.name` is the name of the VolumeSnapshot without pod ordinal and timestamp portion, i.e.: for a VolumeSnapshot name `source-pvc-1560400745`, this field will be `source-pvc`.

* `spec.target.volumeClaimTemplates.spec.storageClassName` indicates the name of the [storageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/) required by the claim.

* `spec.target.volumeClaimTemplates.spec.dataSource` are used only for `VolumeSnapshot`. currently `VolumeSnapshot` is the only supported data source. It will create a new volume and data will be restored to the volume at the same time.
  * `kind` is the type of resource being referenced.

  * `name` is the name of the resource being referenced. In `RestoreSession` crd, The name follows the following convention `${CLAIM_NAME}-<backup session creation timestamp in Unix epoch seconds>`. `${CLAIM_NAME}` are resolved by the Stash operator and replaced by the persistent volume claim name.

Let's create the `RestoreSession` crd we have shown above.

```console
$ kubectl create -f ./docs/examples/volume-snapshot/standalonePVC/restoresession.yaml
restoresession.stash.appscode.com/restore-pvc created
````
Here, restore-pvc is the name of the RestoreSession that has been created.

#### Verify Restored PVC 

If everything goes well, Stash operator creates a Restore Job. Then Restore Job will create new PVC.  

Now, wait a few minutes, you will see that a new PVC with the name `source-pvc` has been created successfully.

check that the status of the PVC is bound

```console
$ kubectl get pvc -n demo source-pvc
NAME         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
source-pvc   Bound    pvc-c2e7ce40-8d98-11e9-bd3e-42010a800011   6Gi        RWO            standard       40s
````
#### Verify Restored Data

Now, We will create a pod to verify whether the restored data has been restored successfully
Below, the YAML for the Pod we are going to create.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: restored-pod
spec:
  containers:
    - name: busybox
      image: busybox
      args:
        - sleep
        - "3600"
      volumeMounts:
        - name: source-data
          mountPath: /demo/data
  volumes:
    - name: source-data
      persistentVolumeClaim:
        claimName: source-pvc
        readOnly: false
```
Let's create the Pod we have shown above.

```console
$ kubectl apply -f ./docs/examples/volume-snapshot/standalonePVC/restored-pod.yaml
pod/restored-pod created
```

Now, wait for Pod to go into the Running state.

```console
$ kubectl get pod -n demo 
NAME           READY   STATUS    RESTARTS   AGE
restored-pod   1/1     Running   0          93s
```

You can check that the `/demo/data/` directory of this pod is populated with data by using the following command,

```console
$ kubectl exec -n demo restored-pod  -- ls -R /demo/data
/demo/data:
lost+found
sample-file.txt
```


### Cleaning Up

To clean up the Kubernetes resources created by this tutorial, run:

```console
$ kubectl delete -n demo pod restored-pod
$ kubectl delete -n demo backupconfiguration pvc-volume-snapshot
$ kubectl delete -n demo restoresession restore-pvc
$ kubectl delete -n demo storageclass standard
$ kubectl delete -n demo volumesnapshotclass default-snapshot-class
```
