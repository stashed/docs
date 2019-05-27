## Backup and Restore Deployment Volume

This Section will describe how to create and restore deployment's volume snapshot using Stash via kubernetes native Api.

To keep everything isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$kubectl create ns demo
namespace/demo created
```
>Note: YAML files used in this tutorial are stored in [/docs/examples/volume-snapshot](/docs/examples/volume-snapshot) directory of [appscode/stash](https://github.com/stashed/stash) repository.

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
kubectl apply -f ./docs/examples/volume-snapshot/storageclass.yaml
storageclass.storage.k8s.io/standard created
```
```console
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