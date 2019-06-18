## Snapshot Deployment's Volumes

This guide will show you how to use Stash to snapshot Deployment's volumes and restore PersistentVolumeClaims from snapshot using Kubernetes [VolumeSnapshot](https://kubernetes.io/docs/concepts/storage/volume-snapshots/) API.

### Before You Begin

* At first, you need to be familiarize with the [GCE Persistent Disk CSI Driver](https://github.com/kubernetes-sigs/gcp-compute-persistent-disk-csi-driver).
* Also need to enable the Kubernetes `VolumeSnapshotDataSource` alpha feature via feature gates.
* You need to have Stash installed in your cluster. 

  If you already don't have Stash installed, please follow this [steps](docs/guides/latest/volumesnapshot/backup.md).

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
The [volumeBindingMode](https://kubernetes.io/docs/concepts/storage/storage-classes/#volume-binding-mode) field controls when volume binding and dynamic provisioning should occur. Kubernetes allows `Immediate` and `WaitForFirstConsumer` modes for binding volumes. The `Immediate` mode indicates that volume binding and dynamic provisioning occurs once the PVC is created and `WaitForFirstConsumer` mode indicates that volume binding and provisioning does not occur until a pod using the PVC is created.

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
* `snapshotter` field to point to the respective CSI driver that is responsible for taking snapshot. For this tutorial, we are using [GCE Persistent Disk CSI Driver](https://github.com/kubernetes-sigs/gcp-compute-persistent-disk-csi-driver). So, we are going to use `pd.csi.storage.gke.io` in this field.

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

This section will show you how to takes snapshot of a Deployment's pvc.

#### Create PersistentVolumeClaim:

At first, let's create PVCs. We will mount this PVCs in our targeted Deployment.

Sample PVCs YAML are given below,

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: backup-pvc-1
  namespace: demo
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 6Gi
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: backup-pvc-2
  namespace: demo
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 6Gi
```
Create the PVCs we have shown above.

```console
$ kubectl apply -f ./pvcs.yaml 
persistentvolumeclaim/backup-pvc-1 created
persistentvolumeclaim/backup-pvc-2 created
```
#### Deploy Deployment:

Now, We will deploy a Deployment that uses the above PVCs. This Deployment will automatically generate sample data (sample-file.txt file) in `/source/data` directory where we have mounted the desired PVCs.

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
      - args: ["touch source/data/sample-file1.txt && sleep 3000"]
        command: ["/bin/sh", "-c"]
        image: busybox
        imagePullPolicy: IfNotPresent
        name: busybox-1
        volumeMounts:
        - mountPath: /source/data
          name: source-data-1
      - args: ["touch source/data/sample-file2.txt && sleep 3000"]
        command: ["/bin/sh", "-c"]
        image: busybox
        imagePullPolicy: IfNotPresent
        name: busybox-2
        volumeMounts:
          - mountPath: /source/data
            name: source-data-2
      restartPolicy: Always
      volumes:
      - name: source-data-1
        persistentVolumeClaim:
         claimName: backup-pvc-1
      - name: source-data-2
        persistentVolumeClaim:
          claimName: backup-pvc-2
```
Let's create the deployment we have shown above.

```console
$ kubectl apply -f ./docs/examples/volume-snapshot/deployment/deployment.yaml
deployment.apps/stash-demo created
```

Now, wait for deployment’s pod to go into the `Running` state.

```console
$ kubectl get pod -n demo 
NAME                          READY   STATUS    RESTARTS   AGE
stash-demo-78c84d5d4b-m4znc   2/2     Running   0          22s
```

Verify that the sample data has been created in `/source/data` directory for `busybox-1` and `busybox-2` container respectively using the following command,

```console
$ kubectl exec -n demo stash-demo-78c84d5d4b-m4znc -c busybox-1 -- ls -R /source/data
/source/data:
lost+found
sample-file1.txt
$ kubectl exec -n demo stash-demo-78c84d5d4b-m4znc -c busybox-2 -- ls -R /source/data
/source/data:
lost+found
sample-file2.txt
```

#### Create BackupConfiguration:

Now, create a `BackupConfiguration` crd to take snapshot of the PVCs of `stash-demo` Deployment.

Sample `BackupConfiguration` crd YAML are given below,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: deployments-volume-snapshot
  namespace: demo
spec:
  schedule: "*/1 * * * *"
  driver: VolumeSnapshotter
  target:
    ref:
      apiVersion: apps/v1
      kind: Deployment
      name: stash-demo
    snapshotClassName: default-snapshot-class
  retentionPolicy:
    name: 'keep-last-5'
    keepLast: 5
    prune: true
```

Here,
* `spec.schedule` is a [cron expression](https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/#schedule) indicates that `BackupSession` will be created every 1 minute.

* `spec.driver` indicates the name of the agent to use to back up the target. Currently, Stash supports `Restic`, `VolumeSnapshotter` drivers. The `VolumeSnapshotter` is used to backup/restore PVC using `VolumeSnapshot` API.

* `spec.target.ref`  refers to the backup target. `apiVersion`, `kind` and `name` refers to the `apiVersion`, `kind` and `name` of the targeted workload respectively. Stash will use this information to create a Volume Snapshotter Job for Creating VolumeSnapshot.

* `spec.target.snapshotClassName` indicates the [VolumeSnapshotClass](https://kubernetes.io/docs/concepts/storage/volume-snapshot-classes/) to use for volume snapshotting.

Let's create the `BackupConfiguration` crd we have shown above.

```console
$ kubectl apply -f ./docs/examples/volume-snapshot/deployment/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/deployments-volume-snapshot created
```
Here, `deployments-volume-snapshot` is the name of the `BackupConfiguration` crd.

 Once the  `BackupConfiguration` crd is created, Stash will create a `CronJob` to take periodic snapshot of `backp-pvc` volume. Stash set the `CronJob` name same as `BackupConfiguration` name. 

 Check that a `CronJob` has been created using the following command,

 ```console
 $ kubectl get cronjob -n demo deployments-volume-snapshot
NAME                          SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
deployments-volume-snapshot   */1 * * * *   False     0        39s             2m41s
 ```

#### Verify Volume Snapshotting 

If everything goes well, The respective `CronJob` creates `BackupSession` crd on each schedule. `BackupSession` crd name will be automatically generated by the following pattern: 
```
<BackupConfiguration name>-<BackupSession creation timestamp in Unix epoch seconds>
``` 
And the label should be set as the following pattern: 
```
backup-configuration:<BackupConfiguration name>
```

Now, wait until a backup schedule appears.

Check that a `BackupSession` has been created using the following command,

```console
$ kubectl get bs -n demo -l  stash.appscode.com/backup-configuration=deployments-volume-snapshot
NAME                                     BACKUPCONFIGURATION           PHASE       AGE
deployments-volume-snapshot-1559026686   deployments-volume-snapshot   Running   26s
```

Stash operator watches for `BackupSession` crd and it will create a volume snapshotter job named `volume-snapshot-<BackupSession name>`.

Verify that the volume snapshotter job has been created using the following command,

```console
$ kubectl get job -n demo volume-snapshot-deployments-volume-snapshot-1559026686
NAMESPACE   NAME                                                     COMPLETIONS   DURATION   AGE
demo        volume-snapshot-deployments-volume-snapshot-1559026686   1/1           19s        40s
```
Finally, volume snapshotter job will create `VolumeSnapshot` crd for the targeted PVCs. The `VolumeSnapshot` names will follow the following pattern:

```
 <PVC name>-<backup session creation timestamp in Unix epoch seconds>
```

Wait a few minutes,  you will see that `backup-pvc-1-1560662826` and `backup-pvc-2-1560662826` VolumeSnapshot has been created Successfully.

```console
$ kubectl get volumesnapshot -n demo
NAME                      AGE
backup-pvc-1-1560662826   69s
backup-pvc-2-1560662826   69s3m
```
You will also see that, The YAML for `backup-pvc-1-1560662826` looks like using the following command,

```console
$ kubectl get volumesnapshot backup-pvc-1-1560662826 -n demo -o yaml
apiVersion: snapshot.storage.k8s.io/v1alpha1
kind: VolumeSnapshot
metadata:
  creationTimestamp: "2019-06-16T05:27:07Z"
  finalizers:
  - snapshot.storage.kubernetes.io/volumesnapshot-protection
  generation: 4
  name: backup-pvc-1-1560662826
  namespace: demo
  resourceVersion: "751269"
  selfLink: /apis/snapshot.storage.k8s.io/v1alpha1/namespaces/demo/volumesnapshots/backup-pvc-1-1560662826
  uid: 6206ec6b-8ff7-11e9-bd3e-42010a800011
spec:
  snapshotClassName: default-snapshot-class
  snapshotContentName: snapcontent-6206ec6b-8ff7-11e9-bd3e-42010a800011
  source:
    apiGroup: null
    kind: PersistentVolumeClaim
    name: backup-pvc-1
status:
  creationTime: "2019-06-16T05:27:08Z"
  readyToUse: true
  restoreSize: 6Gi
```

The snapshots will be stored in the GCE cloud.
You                 can view the snapshot in GCE,
<p align="center">
  <img alt="Stash Backup Flow" src="/docs/images/v1beta1/backends/volumesnapshot/deployment.png">
<figcaption align="center">Fig: Snapshots Screenshot</figcaption>
</p>

Once the snapshotting is completed, `BackupSession` crd phase will be Succeeded. Check the `BackupSession` phase is Succeeded by following command,

```console
$ kubectl get bs -n demo -l  backup-configuration=deployments-volume-snapshot
NAME                                     BACKUPCONFIGURATION           PHASE       AGE
deployments-volume-snapshot-1559026686   deployments-volume-snapshot   Succeeded   40s
```

### Restore PVC from VolumeSnapshot

This section will show you how to restore the PVCs from the snapshots we have taken in earlier section.

#### Create RestoreSession

Create a `RestoreSession` crd to restore PersistentVolumeClaims from snapshot.
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
          name: backup-pvc
        spec:
          accessModes: [ "ReadWriteOnce" ]
          storageClassName: "standard"
          resources:
            requests:
              storage: 6Gi
          dataSource:
            kind: VolumeSnapshot
            name: ${CLAIM_NAME}-1559026686
           # name: backup-pvc-1559026686
            apiGroup: snapshot.storage.k8s.io
```
Here,

* `spec.driver` indicates the name of the agent to use to back up the target. Currently, Stash supports `Restic`, `VolumeSnapshotter` drivers. The `VolumeSnapshotter` is used to backup/restore PVC using `VolumeSnapshot` API.

* `spec.target.volumeClaimTemplates`:
  * `metadata.name` is the name of the restored `PVC` or prefix of the `VolumeSnapshot` name.
  * `spec.dataSource`:
    * `apiGroup` is the group for resource being referenced. Now, kubernetes supports only `snapshot.storage.k8s.io`.
    * `kind` is resource of the kind being referenced. Now, kubernetes supports only `VolumeSnapshot`.
    * `name` is the `VolumeSnapshot` resource name. In `RestoreSession` crd, You can templating the name by using the following convention `${CLAIM_NAME}-<backup session creation timestamp in Unix epoch seconds>`, `${CLAIM_NAME}` is resolved by the Stash operator and replaced by the `metadata.name`. You can set the snapshot name directly.

Let's create the `RestoreSession` crd we have shown above.

```console
$ kubectl create -f ./docs/examples/volume-snapshot/deployment/restoresession.yaml
restoresession.stash.appscode.com/restore-pvc created
````

Here, restore-pvc is the name of the RestoreSession that has been created.

#### Verify Restored PVC 

If everything goes well, Stash will create a `Job` to restore the volumes.  `Job` name will follow the following pattern: `volume-snapshot-<RestoreSession name>`

Check that the `Job` has been created using the following command, 

```console
kubectl get job volume-snapshot-restore-pvc -n demo
NAME                          COMPLETIONS   DURATION   AGE
volume-snapshot-restore-pvc   0/1           7s         7s
```

Wait a few minutes, you will see that a new PVC with the name `backup-pvc` has been created successfully.

check that the status of the PVC is bound,

```console
$ kubectl get pvc -n demo backup-pvc
NAME         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
backup-pvc   Bound    pvc-1963c6c4-81da-11e9-91dc-42010a80001a   6Gi        RWO            standard       27m
````
Once the `Job` is completed, `RestoreSession` crd phase will be Succeeded. Check the `RestoreSession` phase is Succeeded by following command,

```console
$ kubectl get restoresession restore-pvc -n demo
NAME          REPOSITORY-NAME   PHASE       AGE
restore-pvc                     Succeeded   15m
```

#### Verify Restored Data

We will create a new Deployment to verify whether the restored data has been restored successfully.
Below, the YAML for the Deployment we are going to create.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: stash-demo
  name: restore-stash-demo
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
      - args:
          - sleep
          - "3600"
        image: busybox
        imagePullPolicy: IfNotPresent
        name: busybox
        volumeMounts:
        - mountPath: /source/data
          name: source-data-1
      restartPolicy: Always
      volumes:
      - name: source-data-1
        persistentVolumeClaim:
         claimName: backup-pvc-1
```
Let's create the deployment we have shown above.

```console
$ kubectl apply -f ./docs/examples/volume-snapshot/deployment/restored-deployment.yaml
deployment.apps/restore-stash-demo created
```

Now, wait for pod to go into the Running state.

```console
$ kubectl get pod -n demo 
NAME                                  READY   STATUS    RESTARTS   AGE
restore-stash-demo-76cf75b6b8-9c76j   1/1     Running   0          73s
```

You can check that the `/source/data/` directory of this pod is populated with data by using the following command,

```console
$ kubectl exec -n demo restore-stash-demo-76cf75b6b8-9c76j   -- ls -R /source/data
/source/data:
lost+found
sample-file1.txt

```

### Cleaning Up

To clean up the Kubernetes resources created by this tutorial, run:

```console
$ kubectl delete -n demo deployment stash-demo
$ kubectl delete -n demo backupconfiguration deployments-volume-snapshot
$ kubectl delete -n demo restoresession restore-pvc
$ kubectl delete -n demo storageclass standard
$ kubectl delete -n demo volumesnapshotclass default-snapshot-class
```
