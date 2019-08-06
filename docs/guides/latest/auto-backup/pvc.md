---
title: Auto Backup PVC | Stash
description: An step by step guide on how to configure automatic backup for PVCs.
menu:
  product_stash_0.8.3:
    identifier: auto-backup-pvc
    name: Auto Backup for PVCs
    parent: auto-backup
    weight: 30
product_name: stash
menu_name: product_stash_0.8.3
section_menu_id: guides
---

# Auto Backup for PVC

This tutorial will show you how to configure automatic backup for PersistentVolumeClaims. Here, we are going to backup a PVC provisioned from an NFS server using auto-backup.

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [Minikube](https://github.com/kubernetes/minikube).
- Install `Stash` in your cluster following the steps [here](/docs/setup/install.md).
- You will need to have a PVC with `ReadWriteMany` access permission. Here, we are going to use an NFS server to provision a PVC with `ReadWriteMany` access. If you don't have an NFS server running, deploy one by following the guide [here](https://github.com/appscode/third-party-tools/blob/master/storage/nfs/README.md).
- You should be familiar with the following `Stash` concepts:
  - [BackupBlueprint](/docs/concepts/crds/backupblueprint.md/)
  - [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md/)
  - [BackupSession](/docs/concepts/crds/backupsession.md/)
  - [Repository](/docs/concepts/crds/repository.md/)
  - [Function](/docs/concepts/crds/function.md)
  - [Task](/docs/concepts/crds/task.md)

To keep everything isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$ kubectl create ns demo
namespace/demo created
```

>**Note:** YAML files used in this tutorial are stored in  [docs/examples/guides/latest/auto-backup/pvc](/docs/examples/guides/latest/auto-backup/pvc) directory of [stashed/docs](https://github.com/stashed/docs) repository.

**Verify necessary Function and Task:**

Stash uses a `Function-Task` model to automatically backup PVC. When you install Stash, it creates the necessary `Function` and `Task`.

Let's verify that Stash has created the necessary `Function` to backup/restore PVC by the following command,

```console
$ kubectl get function
NAME            AGE
pvc-backup      6h55m
pvc-restore     6h55m
update-status   6h55m
```

Also, verify that the necessary `Task` has been created,

```console
$ kubectl get task
NAME          AGE
pvc-backup    6h55m
pvc-restore   6h55m
```

## Prepare Template

We are going to use [GCS Backend](/docs/guides/latest/backends/gcs.md) to store the backed up data. You can use any supported backend you prefer. You just have to configure Storage Secret and `spec.backend` section of `BackupBlueprint` to match your backend. To know which backed is supported by Stash and how to configure them, please visit [here](/docs/guides/latest/backends/overview.md).

**Create Storage Secret:**

At first, let's create a Storage Secret for the GCS backend,

```console
$ echo -n 'changeit' > RESTIC_PASSWORD
$ echo -n '<your-project-id>' > GOOGLE_PROJECT_ID
$ mv downloaded-sa-json.key > GOOGLE_SERVICE_ACCOUNT_JSON_KEY
$ kubectl create secret generic -n demo gcs-secret \
    --from-file=./RESTIC_PASSWORD \
    --from-file=./GOOGLE_PROJECT_ID \
    --from-file=./GOOGLE_SERVICE_ACCOUNT_JSON_KEY
secret/gcs-secret created
```

**Create BackupBlueprint:**

Now, we have to create a `BackupBlueprint` crd with a template for `Repository` and `BackupConfiguration` object.

Below is the YAML of the `BackupBlueprint` object that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupBlueprint
metadata:
  name: pvc-backup-blueprint
spec:
  # ============== Blueprint for Repository ==========================
  backend:
    gcs:
      bucket: appscode-qa
      prefix: stash-backup/${TARGET_NAMESPACE}/${TARGET_KIND}/${TARGET_NAME}
    storageSecretName: gcs-secret
  # ============== Blueprint for BackupConfiguration =================
  task:
    name: pvc-backup
  schedule: "*/5 * * * *"
  retentionPolicy:
    name: 'keep-last-5'
    keepLast: 5
    prune: true
```

Here,

- `spec.task.name` specifies the `Task` crd name that will be used to backup the targeted PVC.

Note that we have used some variables (format: `${<variable name>}`) in `backend.gcs.prefix` field. Stash will substitute these variables with values from the respective target. To know which variable you can use in this `prefix` field, please visit [here](/docs/concepts/crds/backupblueprint.md#repository-template).

Let's create the `BackupBlueprint` that we have shown above,

```console
$ kubectl apply -f ./docs/examples/guides/latest/auto-backup/pvc/backupblueprint.yaml
backupblueprint.stash.appscode.com/pvc-backup-blueprint created
```

Now, automatic backup is configured for PVC. We just have to add some annotations to the targeted PVC to enable backup.

**Required Annotations for PVC:**

You have to add the following 3 annotations to a targeted PVC to enable backup for it:

1. Name of the `BackupBlueprint` object where a template for `Repository` and `BackupConfiguration` has been defined.

    ```yaml
    stash.appscode.com/backup-blueprint: <BackupBlueprint name>
    ```

2. List of directories that will be backed up. Use comma (`,`) to separate multiple directories. For example, `"/my/target/dir-1,/my/target/dir-2"`.

    ```yaml
    stash.appscode.com/target-paths: "<directories to backup>"
    ```

3. MountPath where the PVC will be mounted inside backup job. This should be same as the directory where the PVC has been mounted inside workload.

    ```yaml
    stash.appscode.com/mountpath: "<mountPath>"
    ```

## Prepare PVC

At first, let's prepare our desired PVC. Here, we are going to create a [PersistentVolume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) (PV) that will use an NFS server as storage. Then, we are going to create a PVC that will bind with the PV. Then, we are going to mount this PVC into a pod. This pod will generate a sample file into the PVC.

**Create PersistentVolume:**

We have deployed an NFS server in `storage` namespace and it is accessible through a Service named `nfs-service`. Now, we are going to create a PV that uses the NFS server as storage.

Below is the YAML of the PV that we are going to create,

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
  labels:
    app: nfs-demo
spec:
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteMany
  nfs:
    server: "nfs-service.storage.svc.cluster.local"
    path: "/"
```

Notice the `metadata.labels` section. Here, we have added `app: nfs-demo` label. We are going to use this label as selector in PVC so that the PVC binds with this PV.

Let's create the PV we have shown above,

```console
$ kubectl apply -f ./docs/examples/guides/latest/auto-backup/pvc/nfs_pv.yaml
persistentvolume/nfs-pv created
```

**Create PersistentVolumeClaim:**

Now, create a PVC to bind with the PV we have just created. Below, is the YAML of the PVC that we are going to create,

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
  namespace: demo
spec:
  accessModes:
  - ReadWriteMany
  storageClassName: ""
  resources:
    requests:
      storage: 1Gi
  selector:
    matchLabels:
      app: nfs-demo
```

Notice the `spec.accessModes` section. We are using `ReadWriteMany` access mode so that multiple pods can use this PVC simultaneously. Without this access mode, Stash will fail to backup the volume if any other pod mounts it during backup.

Also, notice the `spec.selector` section. We have specified `app: nfs-demo` labels as a selector so that it binds with the PV that we have created earlier.

Let's create the PVC we have shown above,

```console
$ kubectl apply -f ./docs/examples/guides/latest/auto-backup/pvc/nfs_pvc.yaml
persistentvolumeclaim/nfs-pvc created
```

Verify that the PVC has bounded with our desired PV,

```console
$ kubectl get pvc -n demo nfs-pvc
NAME      STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
nfs-pvc   Bound    nfs-pv   1Gi        RWX                           61s
```

Here, we can see that the PVC `nfs-pvc` has been bounded with PV `nfs-pv`.

**Generate Sample Data:**

Now, we are going to deploy a sample pod that mounts the PVC we have just created. The pod will generate a sample file named `hello.txt` in `/my/sample/data/` directory. We are going to backup this file using auto-backup.

Below, is the YAML of the pod that we are going to deploy,

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: demo-pod
  namespace: demo
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["/bin/sh", "-c","echo 'hello from sample file.'>>/my/sample/data/hello.txt && sleep 3000"]
    volumeMounts:
    - name: data-volume
      mountPath: /my/sample/data
  volumes:
  - name: data-volume
    persistentVolumeClaim:
      claimName: nfs-pvc
```

Here, we have mount the `nfs-pvc` into `/my/sample/data` directory of the pod. Let's deploy the pod we have shown above,

```console
$ kubectl apply -f ./docs/examples/guides/latest/auto-backup/pvc/pod.yaml
pod/demo-pod created
```

Verify that the pod has generated the sample file,

```console
$ kubectl exec -n demo demo-pod -- cat /my/sample/data/hello.txt
hello from sample file.
```

## Backup

Now, we are going to add auto backup specific annotations to the PVC. Stash watches for PVC with auto-backup annotations. Once it finds a PVC with auto-backup annotations, it will create a `Repository` and a `BackupConfiguration` crd according to respective `BackupBlueprint`. Then, rest of the backup process will proceed as normal backup of a stand-alone PVC as describe [here](/docs/guides/latest/volumes/pvc.md).

**Add Annotations:**

Let's add the auto backup specific annotation to the PVC,

```console
$ kubectl annotate pvc nfs-pvc -n demo --overwrite \
  stash.appscode.com/backup-blueprint=pvc-backup-blueprint \
  stash.appscode.com/target-paths="/my/sample/data" \
  stash.appscode.com/mountpath="/my/sample/data"
```

Verify that the annotations has been added successfully,

```console
$ kubectl get pvc -n demo nfs-pvc -o yaml
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    pv.kubernetes.io/bind-completed: "yes"
    pv.kubernetes.io/bound-by-controller: "yes"
    stash.appscode.com/backup-blueprint: pvc-backup-blueprint
    stash.appscode.com/mountpath: /my/sample/data
    stash.appscode.com/target-paths: /my/sample/data
  name: nfs-pvc
  namespace: demo
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  selector:
    matchLabels:
      app: nfs-demo
  storageClassName: ""
  volumeMode: Filesystem
  volumeName: nfs-pv
status:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi
  phase: Bound
```

Now, Stash will create a `Repository` crd and a `BackupConfiguration` crd according to the template.

**Verify Repository:**

Verify that the `Repository` has been created successfully by the following command,

```console
$ kubectl get repository -n demo
NAME                            INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
persistentvolumeclaim-nfs-pvc
```

If we view the YAML of this `Repository`, we are going to see that the variables `${TARGET_NAMESPACE}`, `${TARGET_KIND}` and `${TARGET_NAME}` has been replaced by `demo`, `presistentvolumeclaim` and `nfs-pvc` respectively.

```console
$ kubectl get repository -n demo persistentvolumeclaim-nfs-pvc -o yaml
```

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: Repository
metadata:
  creationTimestamp: "2019-07-18T09:14:11Z"
  finalizers:
  - stash
  generation: 1
  name: persistentvolumeclaim-nfs-pvc
  namespace: demo
  resourceVersion: "23187"
  selfLink: /apis/stash.appscode.com/v1beta1/namespaces/demo/repositories/persistentvolumeclaim-nfs-pvc
  uid: 67998af5-a93c-11e9-a5e4-0800273e2099
spec:
  backend:
    gcs:
      bucket: appscode-qa
      prefix: stash-backup/demo/persistentvolumeclaim/nfs-pvc
    storageSecretName: gcs-secret
```

**Verify BackupConfiguration:**

Verify that the `BackupConfiguration` crd has been created by the following command,

```console
$ kubectl get backupconfiguration -n demo
NAME                            TASK         SCHEDULE      PAUSED   AGE
persistentvolumeclaim-nfs-pvc   pvc-backup   */5 * * * *            119s
```

Let's check the YAML of this `BackupConfiguration`,

```console
$ kubectl get backupconfiguration -n demo persistentvolumeclaim-nfs-pvc -o yaml
```

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  creationTimestamp: "2019-07-18T09:14:11Z"
  finalizers:
  - stash.appscode.com
  generation: 1
  name: persistentvolumeclaim-nfs-pvc
  namespace: demo
spec:
  repository:
    name: persistentvolumeclaim-nfs-pvc
  retentionPolicy:
    keepLast: 5
    name: keep-last-5
    prune: true
  runtimeSettings: {}
  schedule: '*/5 * * * *'
  target:
    directories:
    - /my/sample/data
    ref:
      apiVersion: v1
      kind: PersistentVolumeClaim
      name: nfs-pvc
    volumeMounts:
    - mountPath: /my/sample/data
      name: stash-volume
  task:
    name: pvc-backup
  tempDir: {}
```

Notice that the `spec.target.ref` is pointing to the `nfs-pvc` PVC. Also, notice that the `spec.target.directories` field has been populated with the information we had provided in `stash.appscode.com/target-directories` annotation.

**Wait for BackupSession:**

Now, wait for the next backup schedule. Run the following command to watch `BackupSession` crd:

```console
$ watch -n 1 kubectl get backupsession -n demo -l=stash.appscode.com/backup-configuration=persistentvolumeclaim-nfs-pvc

Every 1.0s: kubectl get backupsession -n demo ... workstation: Thu Jul 18 15:18:42 2019
NAME                                       BACKUPCONFIGURATION             PHASE       AGE
persistentvolumeclaim-nfs-pvc-1563441309   persistentvolumeclaim-nfs-pvc   Succeeded   3m33s
```

>Note: Respective CronJob creates `BackupSession` crd with the following label `stash.appscode.com/backup-configuration=<BackupConfiguration crd name>`. We can use this label to watch only the `BackupSession` of our desired `BackupConfiguration`.

**Verify Backup:**

When backup session is completed, Stash will update the respective `Repository` to reflect the latest state of backed up data.

Run the following command to check if a snapshot has been sent to the backend,

```console
$ kubectl get repository -n demo persistentvolumeclaim-nfs-pvc
NAME                            INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
persistentvolumeclaim-nfs-pvc   true        41 B   1                3m37s                    5m11s
```

> Stash creates one snapshot for each targeted directory. Since we are taking backup of two directories, two snapshots have been created for this BackupSession.

If we navigate to `stash-backup/demo/persistentvolumeclaim/nfs-pvc` directory of our GCS bucket, we are going to see that the snapshot has been stored there.

<figure align="center">
  <img alt="Backup data of PVC 'nfs-pvc' in GCS backend" src="/docs/images/guides/latest/auto-backup/pvc/pvc_repo.png">
  <figcaption align="center">Fig: Backup data of PVC "nfs-pvc" in GCS backend</figcaption>
</figure>

## Cleanup

To cleanup the Kubernetes resources created by this tutorial, run:

```console
kubectl delete -n demo backupBlueprint/pvc-backup-template
kubectl delete -n demo repository/persistentvolumeclaim-nfs-pvc
kubectl delete -n demo backupconfiguration/persistentvolumeclaim-nfs-pvc

kubectl delete -n demo pod/demo-pod
kubectl delete -n demo pvc/nfs-pvc
kubectl delete -n demo pv/nfs-pv
```

If you would like to uninstall Stash operator, please follow the steps [here](/docs/setup/uninstall.md).
