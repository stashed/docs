## Backup and Restore Deployment Volume

This Section will describe how to backup and restore deployment's volume using Stash via `VolumeSnapshot` and `VolumeSnapshotClass` API resources.

To keep everything isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$ kubectl create ns demo
namespace/demo created
```
>Note: YAML files used in this tutorial are stored in [/docs/examples/volume-snapshot](/docs/examples/volume-snapshot) directory of [appscode/stash](https://github.com/stashed/stash) repository.

#### Create Volume Snapshot

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
$ kubectl apply -f ./docs/examples/volume-snapshot/storageclass.yaml
storageclass.storage.k8s.io/standard created
```
```bash
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

```console
$ kubectl apply -f ./docs/examples/volume-snapshot/deployment.yaml
deployment.apps/stash-demo created
```

Now, wait for deployment’s pod to go into Running state.

```console
$ kubectl get pod -n demo 
NAME                          READY   STATUS    RESTARTS   AGE
stash-demo-5f67c6b4c4-sqgll   1/1     Running    0         17s
```

You can check that the `/source/data/` directory of this pod is populated with data from the `backup-pvc` PVC by using the following command,

```console
$ kubectl exec -n demo stash-demo-7ccd56bf5d-4x27d -- ls -R /source/data
/source/data:
lost+found
sample-file.txt
```

###### Create BackupConfiguration:

Now, we are going to create a `BackupConfiguration` crd for creating Volume Snapshot of `backup-pvc` PVC mounted to `/source/data` directory of `stash-demo` deployment. `BackupConfiguration` crd will create a `BackupSession` crd and start taking periodic backup of `/source/data` directory.

Below, the YAML for `BackupConfiguration` crd we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: deployments-volume-snapshot
  namespace: demo
spec:
  schedule: "*\1 * * * *"
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
* `spec.schedule` is a [cron expression](https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/#schedule) that indicates that `BackupSession` will be created every 1 minute. Then `BackupSession` crd will start taking periodic backup of the data.

* `spec.driver` indicates that BackupConfiguration will be created for creating Volumne Snapshot.

* `spec.target.ref`  refers to the target of backup. `apiVersion`, `kind` and `name` refers to the `apiVersion`, `kind` and `name` of the targeted workload respectively. Stash will use this information to create a Volume Snapshotter Job for Creating Volume Snapshot.

* `spec.target.snapshotClassName` indicates the [VolumeSnapshotClass](https://kubernetes.io/docs/concepts/storage/volume-snapshot-classes/) to use for volume snapshotting.

Before Creating `BackupConfiguration` crd, we need VolumeSnapshotClass.

Below, the YAML for `VolumeSnapshotClass` crd we are going to create,

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

Let's create the `volumeSnapshotClass` and `BackupConfiguration` crd respectively we have shown above.

```console
$ kubectl apply -f ./docs/examples/volume-snapshot/default-volumesnapshotclass.yaml
volumesnapshotclass.snapshot.storage.k8s.io/default-snapshot-class created
```
```console
$ kubectl apply -f ./docs/examples/volume-snapshot/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/deployments-volume-snapshot created
```
here, `deployments-volume-snapshot` is the name of the `BackupConfiguration` crd that has been created successfully.

###### Verify Creating Volume Snapshot 

if everything goes well, Stash will create a `BackupSession` crd with CronJob schedule. `BackupSession` crd name will be automatically generated by the following pattern `<BackupConfiguration name>-<creation timestamp in unix epoch seconds>`

Let's check the `BackupSession` crd that will be created after schedule time

```console
$ kubectl get backupsession -n demo | grep "deployments-volume-snapshot"
deployments-volume-snapshot-1559026686   deployments-volume-snapshot   Running   26s
```
here, `deployments-volume-snapshot-1559026686` is the name of the `BackupSession` crd that has been created sucessfully after 1 minute schedule time.

if we check the status of the `BackupSession` then we will see that the phase of backupsession status is `Running`. let's check it by using the following command

```console
$ kubectl describe bs -n  demo       deployments-volume-snapshot-1559026686 
Name:         deployments-volume-snapshot-1559026686
Namespace:    demo
...
...
API Version:  stash.appscode.com/v1beta1
Kind:         BackupSession
Metadata:
  .....
  .....
Spec:
  Backup Configuration:
    Name:  deployments-volume-snapshot
Status:
  Phase:        Running
  Total Hosts:  1
Events:
  .....
  .....

```

`BackupSession` crd will create volume snapshotter job, job name follows the following pattern `volume-snapshot-<BackupSession name>`. let's check it by using the command


```console
$ kubectl get job -n demo volume-snapshot-deployments-volume-snapshot-1559026686
NAMESPACE   NAME                                                     COMPLETIONS   DURATION   AGE
demo        volume-snapshot-deployments-volume-snapshot-1559026686   1/1           19s        40s
```

Finally, snapshotter job will create volume snapshot vai kubernetes native API. `volumesnapshot` name will be formed by the following pattern `<PVC name>-<backup session creation timestamp in unix epoch seconds>`.

After waiting sometimes, we will check for volume snapshot by using the following command.  

```console
$ kubectl get volumesnapshot -n demo backup-pvc-1559026686
NAMESPACE   NAME                           AGE
demo        backup-pvc-1559026686          153m

```
here, `backup-pvc-1559026686` is the name of the volume snapshot that has been created successfully. if we check the status of the `BackupSession` crd then we will see that the phase of backupsession status is `Succeeded`. let's check it by using the following command

```console
$ kubectl describe bs -n  demo       deployments-volume-snapshot-1559026686 
Name:         deployments-volume-snapshot-1559026686
Namespace:    demo
....
....
API Version:  stash.appscode.com/v1beta1
Kind:         BackupSession
Metadata:
  ...
  ...
Spec:
  Backup Configuration:
    Name:  deployments-volume-snapshot
Status:
  Phase:             Succeeded
  Session Duration:  1m17.858810867s
  Stats:
    ...
    ...
  Total Hosts:  1
Events:
  ....
  ...
```

#### Restore from Volume Snapshot





