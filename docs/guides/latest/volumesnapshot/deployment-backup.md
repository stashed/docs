## Backup Deployment's Volumes

This tutorial will show you how to use Stash Operator to backup Deployment's volumes using [Volume Snapshot](https://kubernetes.io/docs/concepts/storage/volume-snapshots/) API resources.

##### Before You Begin

* At first, you need to familiarize with the GCE CSI-driver, to enable the Volume Snapshotting feature via feature gate and to install Stah Operator in your cluster. If you are not familiar with that please visit [here](docs/guides/latest/volumesnapshot/backup.md) 

To keep everything isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$ kubectl create ns demo
namespace/demo created
```

Now, we will create a StorageClass for further uses. 
Sample StorageClass YAML are given below,

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

Let's create the StorageClass we have shown above.

```console
$ kubectl apply -f ./docs/examples/volume-snapshot/storageclass.yaml
storageclass.storage.k8s.io/standard created
```
We will also create a `VolumeSnapshotClass` for further uses.

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



>Note: YAML files used in this tutorial are stored in [/docs/examples/volume-snapshot](/docs/examples/volume-snapshot) directory of [appscode/stash](https://github.com/stashed/stash) repository.


#### Create Volume Snapshot

In order to create Volume Snapshot, At first, we will create a PVC that holds some sample data.

Sample PVC YAML are given below,

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
Let's create the PVC we have shown above.

```console
kubectl apply -f ./docs/examples/volume-snapshot/pvc.yaml
persistentvolumeclaim/backup-pvc created
```
##### Deploy Workload:

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

Let's create the deployment we have shown above.

```console
$ kubectl apply -f ./docs/examples/volume-snapshot/deployment.yaml
deployment.apps/stash-demo created
```

Now, wait for deployment’s pod to go into the Running state.

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

##### Create BackupConfiguration:

Now, we are going to create a `BackupConfiguration` crd for creating Volume Snapshot of `backup-pvc` PVC mounted to `/source/data` directory of `stash-demo` deployment. `BackupConfiguration` crd will create a `BackupSession` crd and start taking periodic backup of `/source/data` directory.

Sample `BackupConfiguration` crd YAML are given below,

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
* `spec.schedule` is a [cron expression](https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/#schedule) that indicates that `BackupSession` will be created every 1 minute. Then `BackupSession` crd will start taking the periodic backup of the data.

* `spec.driver` The driver indicates the name of the agent to use to back up the target. Supported values are `Restic`, `VolumeSnapshotter`. `VolumeSnapshotter` are used for creating/restoring volume snapshot.

* `spec.target.ref`  refers to the backup target. `apiVersion`, `kind` and `name` refers to the `apiVersion`, `kind` and `name` of the targeted workload respectively. Stash will use this information to create a Volume Snapshotter Job for Creating Volume Snapshot.

* `spec.target.snapshotClassName` indicates the [VolumeSnapshotClass](https://kubernetes.io/docs/concepts/storage/volume-snapshot-classes/) to use for volume snapshotting.


Let's create the `BackupConfiguration` crd we have shown above.




```console
$ kubectl apply -f ./docs/examples/volume-snapshot/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/deployments-volume-snapshot created
```
here, `deployments-volume-snapshot` is the name of the `BackupConfiguration` crd that has been created successfully.

##### Verify Creating Volume Snapshot 

If everything goes well, the Stash operator will create a `BackupSession` crd with the CronJob schedule. `BackupSession` crd name will be automatically generated by the following pattern `<BackupConfiguration name>-<creation timestamp in Unix epoch seconds>` and Label should be set as the following pattern `backup-configuration:<BackupConfiguration name>`.

Now, wait a few minutes,  you will see that `BackupSession` has been created.
Check that the phase of `BackupSession` status is `Running` 

```console
$ kubectl get bs -n demo -l  backup-configuration=deployments-volume-snapshot
NAME                                     BACKUPCONFIGURATION           PHASE       AGE
deployments-volume-snapshot-1559026686   deployments-volume-snapshot   Running   26s
```
here, `deployments-volume-snapshot-1559026686` is the name of the `BackupSession` crd that has been created successfully after 1 minute.


`BackupSession` crd will create volume snapshotter job, job name follows the following pattern `volume-snapshot-<BackupSession name>`. 
Check it by using the following command


```console
$ kubectl get job -n demo volume-snapshot-deployments-volume-snapshot-1559026686
NAMESPACE   NAME                                                     COMPLETIONS   DURATION   AGE
demo        volume-snapshot-deployments-volume-snapshot-1559026686   1/1           19s        40s
```

Finally, snapshotter job will create volume snapshot vai Kubernetes native API. `VolumeSnapshot` name will be formed by the following pattern `<PVC name>-<backup session creation timestamp in Unix epoch seconds>`.

Wait a few minutes,  you will see that `backup-pvc-1559026686` Volume Snapshot has been created Successfully.

```console
$ kubectl get volumesnapshot -n demo backup-pvc-1559026686
NAMESPACE   NAME                           AGE
demo        backup-pvc-1559026686          153m

```
If you check the status of the `BackupSession` crd then we will see that the phase of backupsession status is `Succeeded`. 

```console
$ kubectl get bs -n demo -l  backup-configuration=deployments-volume-snapshot
NAME                                     BACKUPCONFIGURATION           PHASE       AGE
deployments-volume-snapshot-1559026686   deployments-volume-snapshot   Running   40s
```
##### Cleaning Up

To clean up the Kubernetes resources created by this tutorial, run:

```console
$ kubectl delete -n demo deployment stash-demo
$ kubectl delete -n demo backupconfiguration deployments-volume-snapshot
$ kubectl delete -n demo storageclass standard
$ kubectl delete -n demo volumesnapshotclass default-snapshot-class
```


