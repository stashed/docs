## Restore Volumes for Deployment

This tutorial will show you how to use Stash Operator to restore volumes from VolumeSnapshot using Volume Snapshot API resources.

##### Before You Begin

* At first, you need to familiarize with the GCE CSI-driver, to enable the Volume Snapshotting feature via feature gate and to install Stah Operator in your cluster. If you are not familiar with that please visit [here](docs/guides/latest/volumesnapshot/backup.md)

*  Then, you need to have VolumeSnapshot taken by Stash. If you already don’t have any VolumeSnapshot, create one by following this [backup tutorial](docs/guides/latest/volumesnapshot/deployment-backup.md).

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

##### Create Restore Session for restore volume

Now we are going to create `RestoreSession` crd for restoring data from Volume Snapshot.

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
            apiGroup: snapshot.storage.k8s.io
```
Here,

* `spec.driver` Driver indicates the name of the agent to use to backup the target. Supported values are `Restic`, `VolumeSnapshotter`. `VolumeSnapshotter` are used for creating/restoring volume snapshot.

* `spec.target.volumeClaimTemplates.spec.storageClassName` indicates the name of the [storageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/) required by the claim.

* `spec.target.volumeClaimTemplates.spec.dataSource` are used only for `VolumeSnapshot`. currently `VolumeSnapshot` is the only supported data source. It will create a new volume and data will be restored to the volume at the same time.
  * `kind` Kind is the type of resource being referenced.

  * `name` Name is the name of resource being referenced. In `RestoreSession` crd, The name follows the following convension `${CLAIM_NAME}-<backup session creation timestamp in unix epoch seconds>`. `${CLAIM_NAME}` are resolved by the Stash operator and replaced by the persistent volume claim name.

Let's create the `RestoreSession` crd we have shown avobe.

```console
$ kubectl create -f ./docs/examples/volume-snapshot/restoresession.yaml
restoresession.stash.appscode.com/restore-pvc created
````
here, restore-pvc is the name of the new pvc name that has been created successfully.

##### Verify Restoring data from  Volume Snapshot

If everything goes well, Stash operator will create Restore Job. Then Restore Job will create new PVC.  

Now, wait a few minutes, you will see that a new PVC with the name `backup-pvc` has been created.

check that the status of the PVC is bound

```console
$ kubectl get pvc -n demo backup-pvc
NAME         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
backup-pvc   Bound    pvc-1963c6c4-81da-11e9-91dc-42010a80001a   6Gi        RWO            standard       27m
````

##### Cleaning Up

To cleanup the Kubernetes resources created by this tutorial, run:

```console
$ kubectl delete -n demo restoresession restore-pvc
$ kubectl delete -n demo storageclass standard
```
