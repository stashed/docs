## Snapshot Deployment's Volumes

This guide will show you how to use Stash to snapshot Deployment's volumes and restore them from snapshot using Kubernetes [VolumeSnapshot](https://kubernetes.io/docs/concepts/storage/volume-snapshots/) API. In this guide, we are going to backup the volumes in Google Cloud Storage with the help of [GCE Persistent Disk CSI Driver](https://github.com/kubernetes-sigs/gcp-compute-persistent-disk-csi-driver).

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

Here, we are going to deploy a Deployment with two PVCs and generate some sample data on it. Then, we will take snapshot of those PVCs using Stash.

#### Create PersistentVolumeClaim:

At first, let's create two sample PVCs. We will mount these PVCs in our targeted Deployment.

Below is the YAML of the sample PVCs,

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: source-pvc-1
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
  name: source-pvc-2
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
persistentvolumeclaim/source-pvc-1 created
persistentvolumeclaim/source-pvc-2 created
```
#### Deploy Deployment:

Now, we will deploy a Deployment that uses the above PVCs. This Deployment will automatically generate sample data (sample-file.txt file) in `/source/data` directory where we have mounted the desired PVCs.

Below is the YAML of the Deployment that we are going to create,

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
         claimName: source-pvc-1
      - name: source-data-2
        persistentVolumeClaim:
          claimName: source-pvc-2
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

Below is the YAML of the `BackupConfiguration` crd that we are going to create,

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
* `spec.schedule` is a [cron expression](https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/#schedule) indicates that `BackupSession` will be created at 1 minute interval.

* `spec.driver` indicates the name of the agent to use to back up the target. Currently, Stash supports `Restic`, `VolumeSnapshotter` drivers. The `VolumeSnapshotter` is used to backup/restore PVC using `VolumeSnapshot` API.

* `spec.target.ref`  refers to the backup target. `apiVersion`, `kind` and `name` refers to the `apiVersion`, `kind` and `name` of the targeted workload respectively. Stash will use this information to create a Volume Snapshotter Job for creating VolumeSnapshot.

* `spec.target.snapshotClassName` indicates the [VolumeSnapshotClass](https://kubernetes.io/docs/concepts/storage/volume-snapshot-classes/) to use for volume snapshotting.

Let's create the `BackupConfiguration` crd we have shown above.

```console
$ kubectl apply -f ./docs/examples/volume-snapshot/deployment/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/deployments-volume-snapshot created
```

#### Verify CronJob:

If everything goes well, Stash will create a `CronJob` to take periodic snapshot of `source-pvc-1` and `source-pvc-2` volumes of the deployment with the schedule specified in `spec.schedule` field of `BackupConfiguration` crd.

 Check that the `CronJob` has been created using the following command,

 ```console
 $ kubectl get cronjob -n demo 
NAME                          SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
deployments-volume-snapshot   */1 * * * *   False     0        39s             2m41s
 ```

#### Wait for BackupSession:

The `deployments-volume-snapshot` CronJob will trigger a backup on each schedule by creating a `BackpSession` crd.
 
Wait for a schedule to appear. Run the following command to watch `BackupSession` crd,

```console
$ watch -n 1 kubectl get backupsession -n demo
Every 1.0s: kubectl get backupsession -n demo                      suaas-appscode: Tue Jun 18 18:35:41 2019

NAME                                     BACKUPCONFIGURATION           PHASE       AGE
deployments-volume-snapshot-1559026686   deployments-volume-snapshot   Running   5s
deployments-volume-snapshot-1559026686   deployments-volume-snapshot   Succeeded   50s
```
We can see above that the backup session has succeeded. Now, we will verify that the `VolumeSnapshot` has been created and the snapshots has been stored in the respective backend.

#### Verify Volume Snapshotting and Backup:

Once a `BackupSession` crd is created, it creates volume snapshotter `Job`. Then the `Job` creates `VolumeSnapshot` crd for the targeted PVCs.The `VolumeSnapshot` name follows the following pattern:

```
 <PVC name>-<backup session creation timestamp in Unix epoch seconds>
```

Check that the `VolumeSnapshot` has been created Successfully.

```console
$ kubectl get volumesnapshot -n demo
NAME                      AGE
source-pvc-1-1560662826   69s
source-pvc-2-1560662826   72s
```
We will also see that, The YAML for `source-pvc-1-1560662826` looks like using the following command,

```console
$ kubectl get volumesnapshot source-pvc-1-1560662826 -n demo -o yaml
apiVersion: snapshot.storage.k8s.io/v1alpha1
kind: VolumeSnapshot
metadata:
  creationTimestamp: "2019-06-16T05:27:07Z"
  finalizers:
  - snapshot.storage.kubernetes.io/volumesnapshot-protection
  generation: 4
  name: source-pvc-1-1560662826
  namespace: demo
  resourceVersion: "751269"
  selfLink: /apis/snapshot.storage.k8s.io/v1alpha1/namespaces/demo/volumesnapshots/source-pvc-1-1560662826
  uid: 6206ec6b-8ff7-11e9-bd3e-42010a800011
spec:
  snapshotClassName: default-snapshot-class
  snapshotContentName: snapcontent-6206ec6b-8ff7-11e9-bd3e-42010a800011
  source:
    apiGroup: null
    kind: PersistentVolumeClaim
    name: source-pvc-1
status:
  creationTime: "2019-06-16T05:27:08Z"
  readyToUse: true
  restoreSize: 6Gi
```

Now, if we navigate to the `Snapshots` in the GCS navigation menu, we will see snapshots has been stored successfully.

<p align="center">
  <img alt="Stash Backup Flow" src="/docs/images/v1beta1/backends/volumesnapshot/deployment.png">
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
    volumeClaimTemplates:
      - metadata:
          name: source-pvc-1
        spec:
          accessModes: [ "ReadWriteOnce" ]
          storageClassName: "standard"
          resources:
            requests:
              storage: 6Gi
          dataSource:
            kind: VolumeSnapshot
            name: ${CLAIM_NAME}-1560662826
            # name: source-pvc-1-1560662826
            apiGroup: snapshot.storage.k8s.io
      - metadata:
          name: source-pvc-2
        spec:
          accessModes: [ "ReadWriteOnce" ]
          storageClassName: "standard"
          resources:
            requests:
              storage: 6Gi
          dataSource:
            kind: VolumeSnapshot
            name: ${CLAIM_NAME}-1560662826
            # name: source-pvc-2-1560662826
            apiGroup: snapshot.storage.k8s.io
```
Here,

* `spec.target.volumeClaimTemplates`:
  * `metadata.name` is the name of the restored `PVC` or prefix of the `VolumeSnapshot` name.
  * `spec.dataSource`: `spec.dataSource` specifies the source of the data from where the newly created PVC will be intialized. It requires following fields to set:
    * `apiGroup` is the group for resource being referenced. Now, kubernetes supports only `snapshot.storage.k8s.io`.
    * `kind` is resource of the kind being referenced. Now, kubernetes supports only `VolumeSnapshot`.
    * `name` is the `VolumeSnapshot` resource name. In `RestoreSession` crd, You can template the name by using the following convention `${CLAIM_NAME}-<backup session creation timestamp in Unix epoch seconds>`, `${CLAIM_NAME}` is resolved by the Stash operator and replaced by the `metadata.name`. You can set the snapshot name directly.

Let's create the `RestoreSession` crd we have shown above.

```console
$ kubectl create -f ./docs/examples/volume-snapshot/deployment/restoresession.yaml
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

#### Verify Restored PVC 

Once a restore process is complete, we will see that new PVCs with the name `source-pvc-1` and `source-pvc-2` has been created successfully.

check that the status of the PVCs are bound,

```console
$ kubectl get pvc -n demo
NAME           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
source-pvc-1   Bound    pvc-e32b18f6-91c0-11e9-bd3e-42010a800011   6Gi        RWO            standard       30s
source-pvc-2   Bound    pvc-e32db936-91c0-11e9-bd3e-42010a800011   6Gi        RWO            standard       30s
````

#### Verify Restored Data

We will create a new Deployment to verify whether the restored data has been restored successfully.

Below is the YAML of the Deployment that we are going to create,

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
        - args:
            - sleep
            - "3600"
          image: busybox
          imagePullPolicy: IfNotPresent
          name: busybox-1
          volumeMounts:
            - mountPath: /source/data
              name: source-data-1
        - args:
            - sleep
            - "3600"
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
            claimName: source-pvc-1
        - name: source-data-2
          persistentVolumeClaim:
            claimName: source-pvc-2 
```
Let's create the deployment we have shown above.

```console
$ kubectl apply -f ./docs/examples/volume-snapshot/deployment/restored-deployment.yaml
deployment.apps/restore-stash-demo created
```

Now, wait for deployment’s pod to go into the `Running` state.

```console
$ kubectl get pod -n demo 
NAME                                  READY   STATUS    RESTARTS   AGE
restore-stash-demo-76cf75b6b8-9c76j   2/2     Running   0          73s
```

Verify that the sample data has been created in `/source/data` directory for `busybox-1` and `busybox-2` container respectively using the following command,

```console
$ kubectl exec -n demo stash-demo-76cf75b6b8-9c76j -c busybox-1 -- ls -R /source/data
/source/data:
lost+found
sample-file1.txt
$ kubectl exec -n demo stash-demo-76cf75b6b8-9c76j -c busybox-2 -- ls -R /source/data
/source/data:
lost+found
sample-file2.txt
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
