---
title: Cross-Cluster Backup and Restore | Stash
description: A guide on how to use backup and restore across clusters using Stash.
menu:
  docs_{{ .version }}:
    identifier: advance-use-case-cross-cluster-backup
    name: Cross-Cluster Backup and Restore
    parent: advance-use-case
    weight: 60
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: guides
---

# Cross-Cluster Backup and Restore

This guide will show you how to take backup and restore across clusters using Stash.

## Before You Begin

- At first, you need to have running Kubernetes clusters, and the `kubectl` command-line tool must be configured to communicate with your clusters. We will use kind clusters throughout this tutorial. To know more about kind clusters, follow this doc [here](https://kind.sigs.k8s.io/docs/user/quick-start/).

- You should be familiar with the following `Stash` concepts:
  - [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md)
  - [BackupSession](/docs/concepts/crds/backupsession.md)
  - [RestoreSession](/docs/concepts/crds/restoresession.md)
  - [Repository](/docs/concepts/crds/repository.md)

## Backup from `prod` Cluster

To demonstrate the cross-clusters backup and restore capabilities, we will use the `prod` cluster for taking backup and restoring the backup into the `staging` cluster.

### Prepare Cluster

Let's create a cluster named `prod`,

```bash
$ kind create cluster --name prod
Creating cluster "prod" ...
 ✓ Ensuring node image (kindest/node:v1.23.0) 🖼
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-prod"
You can now use your cluster with:

kubectl cluster-info --context kind-prod

Have a nice day! 👋
```

To verify your current cluster,

```bash
$ kubectl config current-context
kind-prod
```

if you are currently not in the `prod` cluster, you can switch your cluster by the following command,

```bash
$ kubectl config use-context kind-prod
Switched to context "kind-prod".
```

We are going to create a namespace named `demo` in the `prod` cluster,

```bash
$ kubectl create ns demo
namespace/demo created
```

>**Note:** YAML files used in this tutorial can be found [here](https://github.com/stashed/docs/guides/advanced-use-case/cross-cluster-backup/examples).

Install `Stash` in your `prod` cluster following the steps [here](/docs/setup/README.md).

### Deploy Workload

Let's deploy a Deployment and an associated PVC in the `prod` cluster at the beginning. This Deployment will automatically generate sample data in the `/source/data` directory.

Below are the YAMLs of the Deployment and PVC that we are going to create,

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: stash-sample-data
  namespace: demo
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: stash-demo
  name: stash-demo
  namespace: demo
spec:
  replicas: 3
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
      - args: ["echo sample_data > /source/data/data.txt && sleep 3000"]
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
          claimName: stash-sample-data
```

Let's create the Deployment and PVC we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/advanced-use-case/cross-cluster-backup/examples/deployment_prod.yaml
persistentvolumeclaim/stash-sample-data created
deployment.apps/stash-demo created
```

Now, wait for the pods of the Deployment to go into the `Running` state. You can verify the `Status` of the pods by executing the following command,

```bash
$ kubectl get pods -n demo
NAME                          READY   STATUS    RESTARTS   AGE
stash-demo-7678679bcb-2s927   1/1     Running   0          29s
stash-demo-7678679bcb-b849l   1/1     Running   0          29s
stash-demo-7678679bcb-p62vn   1/1     Running   0          29s
```

Verify that the sample data has been created in `/source/data` directory using the following command,

```bash
$ kubectl exec -n demo stash-demo-7678679bcb-2s927 -- cat /source/data/data.txt
sample_data
```

### Prepare Backend

We are going to store our backed-up data into a GCS bucket. We have to create a Secret with the necessary credentials and a Repository CRD to use this backend. 

If you want to use a different backend, please read the respective backend configuration doc [here](/docs/guides/backends/overview.md).

> For GCS backend, if the bucket does not exist, Stash needs `Storage Object Admin` role permissions to create the bucket. For more details, please check the following [guide](/docs/guides/backends/gcs.md).

**Create Secret:**

Let's create a secret called `gcs-secret` in `demo` namespace with access credentials to our desired GCS bucket,

```bash
$ echo -n 'changeit' > RESTIC_PASSWORD
$ echo -n '<your-project-id>' > GOOGLE_PROJECT_ID
$ cat /path/to/downloaded-sa-key.json > GOOGLE_SERVICE_ACCOUNT_JSON_KEY
$ kubectl create secret generic -n demo gcs-secret \
    --from-file=./RESTIC_PASSWORD \
    --from-file=./GOOGLE_PROJECT_ID \
    --from-file=./GOOGLE_SERVICE_ACCOUNT_JSON_KEY
secret/gcs-secret created
```

Now, we are ready to backup our workload's data to our desired backend.

**Create Repository:**

Now, create a `Repository` using this secret. Below is the YAML of the `Repository` object we are going to create, 

```yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: gcs-repo
  namespace: demo
spec:
  backend:
    gcs:
      bucket: stash-testing
      prefix: /cross-cluster/deployment/sample-deployment
    storageSecretName: gcs-secret
```

Let's create the Repository we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/advanced-use-case/cross-cluster-backup/examples/repository_prod.yaml
repository.stash.appscode.com/gcs-repo created
```

Now, we are ready to backup our sample data into this backend.

### Configure Backup

We are going to create a `BackupConfiguration` object in the `demo` namespace targeting the `stash-demo` Deployment that we have deployed earlier. This `BackupConfiguration` will refer to the `gcs-repo` repository. Stash will inject a sidecar container into the target. It will also create a `CronJob` to take periodic backup of the `/source/data` directory of the target.

**Create BackupConfiguration:**

Below is the YAML of the `BackupConfiguration` crd that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: deployment-backup
  namespace: demo
spec:
  repository:
    name: gcs-repo
  schedule: "*/5 * * * *"
  target:
    ref:
      apiVersion: apps/v1
      kind: Deployment
      name: stash-demo
    volumeMounts:
    - name: source-data
      mountPath: /source/data
    paths:
    - /source/data
  retentionPolicy:
    name: 'keep-last-5'
    keepLast: 5
    prune: true
```

Here,

- `spec.repository.name` refers to the Repository object that holds backend information.
- `spec.schedule` is a cron expression that indicates BackupSession will be created at 5 minutes intervals.
- `spec.target.ref` refers to the `stash-demo` Deployment.
- `spec.target.volumeMounts` specifies a list of volumes and their mountPath that contain the target paths.
- `spec.target.paths` specifies list of file paths to backup.

Let's create the `BackupConfiguration` object we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/advanced-use-case/cross-cluster-backup/examples/backupconfiguration_prod.yaml
backupconfiguration.stash.appscode.com/deployment-backup created
```

**Verify BackupConfiguration Ready:**

If everything goes well, the Phase of the BackupConfiguration should be `Ready`. Let's check the BackupConfiguration phase,

```bash
❯ kubectl get backupconfiguration -n demo

NAME                TASK   SCHEDULE      PAUSED   PHASE   AGE
deployment-backup          */5 * * * *            Ready   48s
```

**Verify CronJob:**

Stash will also create a `CronJob` with the schedule specified in the `spec.schedule` field of the `BackupConfiguration` crd.

Verify that the `CronJob` has been created using the following command,

```bash
$ kubectl get cronjob -n  prod
NAME                              SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
stash-trigger-deployment-backup   */5 * * * *   False     0        28s             2m14s
```

### Verify Backup

The `stash-trigger-deployment-backup` CronJob will trigger a backup on each scheduled slot by creating a `BackupSession` crd. The sidecar container watches for the `BackupSession` object. When it finds one, it will take backup immediately.

Wait for the next schedule for the backup. Run the following command to watch `BackupSession` crd,

```bash
$ kubectl get backupsession -n prod -w

NAME                           INVOKER-TYPE          INVOKER-NAME        PHASE      DURATION   AGE
deployment-backup-1647238200   BackupConfiguration   deployment-backup   Running               5s
deployment-backup-1647238200   BackupConfiguration   deployment-backup   Running               16s
deployment-backup-1647238200   BackupConfiguration   deployment-backup   Running               27s
deployment-backup-1647238200   BackupConfiguration   deployment-backup   Succeeded   28s       27s
```

We can see from the above output that the backup session has succeeded.

## Restore into `staging` Cluster

This section will demonstrate restoring the backed-up data into the `staging` cluster.

**Stop Taking Backup of the Old Deployment:**

At first, let's stop taking any further backup of the old Deployment so that no backup is taken during the restore process. We are going to pause the `BackupConfiguration` that we created earlier. Then, Stash will stop taking any further backup for this Deployment. You can learn more how to pause a scheduled backup [here](/docs/guides/advanced-use-case/pause-backup.md)

Let's pause the `deployment-backup` BackupConfiguration,

```bash
$ kubectl patch backupconfiguration -n demo deployment-backup --type="merge" --patch='{"spec": {"paused": true}}'
backupconfiguration.stash.appscode.com/deployment-backup patched
```

You can also use the Stash kubectl plugin to pause the backup like the following,

```bash
$ kubectl stash pause backup --backupconfig=deployment-backup -n demo
BackupConfiguration demo/deployment-backup has been paused successfully.
```

Verify that the BackupConfiguration has been paused,

```bash
$ kubectl get backupconfiguration -n demo
NAME                TASK   SCHEDULE      PAUSED   PHASE   AGE
deployment-backup          */5 * * * *   true     Ready   49m
```

Notice the `PAUSED` column. Value `true` indicates that the BackupConfiguration has been paused.

### Prepare Cluster

Let's create a cluster named `staging`,

```bash
$ kind create cluster --name staging
Creating cluster "staging" ...
 ✓ Ensuring node image (kindest/node:v1.23.0) 🖼
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-staging"
You can now use your cluster with:

kubectl cluster-info --context kind-staging

Have a nice day! 👋
```

To verify the current cluster you are working on,

```bash
$ kubectl config current-context
kind-staging
```

If you are currently not in the `staging` cluster, you can switch your cluster by the following command,

```bash
$ kubectl config use-context kind-staging
Switched to context "kind-staging".
```

We are going to create a namespace named `demo` in the `staging` cluster,

```bash
$ kubectl create ns demo
namespace/demo created
```

Install `Stash` in your `staging` cluster following the steps [here](/docs/setup/README.md).

### Deploy Restore Workload

We are going to create a new Deployment named `stash-recovered` and a PVC as a storage of the Deployment. We will restore the backed-up data inside it.

Here is the YAML of the PVC and the Deployment,

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: demo-pvc
  namespace: demo
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: stash-recovered
  name: stash-recovered
  namespace: demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: stash-recovered
  template:
    metadata:
      labels:
        app: stash-recovered
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
          name: source-data
      restartPolicy: Always
      volumes:
      - name: source-data
        persistentVolumeClaim:
          claimName: demo-pvc
```

Let's create the Deployment we have shown above.

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/advanced-use-case/cross-cluster-backup/examples/deployment_staging.yaml
persistentvolumeclaim/demo-pvc created
deployment.apps/stash-recovered created
```

### Prepare Backend Info

We have to re-create the Repository CR and the respective Secret that we have used in the `prod` cluster.

**Create Secret:**

Let's create a secret called `gcs-secret` in `demo` namespace with access credentials to our GCS bucket,

```bash
$ echo -n 'changeit' > RESTIC_PASSWORD
$ echo -n '<your-project-id>' > GOOGLE_PROJECT_ID
$ cat /path/to/downloaded-sa-key.json > GOOGLE_SERVICE_ACCOUNT_JSON_KEY
$ kubectl create secret generic -n demo gcs-secret \
    --from-file=./RESTIC_PASSWORD \
    --from-file=./GOOGLE_PROJECT_ID \
    --from-file=./GOOGLE_SERVICE_ACCOUNT_JSON_KEY
secret/gcs-secret created
```

**Create Repository:**

Now, create a `Repository` using this secret. Below is the YAML of `Repository` crd we are going to create, 

```yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: gcs-repo
  namespace: demo
spec:
  backend:
    gcs:
      bucket: stash-testing
      prefix: /cross-cluster/deployment/sample-deployment
    storageSecretName: gcs-secret
```

Let's create the Repository we have shown above,
```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/advanced-use-case/cross-cluster-backup/examples/repository_prod.yaml
repository.stash.appscode.com/gcs-repo created
```

Now, we are ready to restore our sample data from this specified backend.

### Configure Restore

Now, we need to create a `RestoreSession` object targeting the `stash-recovered` Deployment to restore the backed-up data inside it.

**Create RestoreSession:**

Below is the YAML of the `RestoreSesion` object that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: deployment-restore
  namespace: demo
spec:
  repository:
    name: gcs-repo
  target: # target indicates where the recovered data will be stored
    ref:
      apiVersion: apps/v1
      kind: Deployment
      name: stash-recovered
    volumeMounts:
    - name:  source-data
      mountPath:  /source/data
    rules:
    - paths:
      - /source/data/
```

Here,

- `spec.repository.name` specifies the name of the Repository.
- `spec.target.ref` refers to the target workload where the recovered data will be stored.
- `spec.target.volumeMounts` specifies a list of volumes and their mountPath where the data will be restored.
  - `mountPath` must be the same mountPath as the original volume because Stash stores the absolute path of the backed-up files. If you use a different mountPath for the restored volume the backed up files will not be restored into your desired volume.

Let's create the `RestoreSession` crd we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/advanced-use-case/cross-cluster-backup/examples/restoresession_staging.yaml
restoresession.stash.appscode.com/deployment-restore created
```

Now, wait for the RestoreSession phase to go into `Succeeded` state.

**Wait for RestoreSession to Succeeded:**

Run the following command to watch the RestoreSession phase,

```bash
$ kubectl get restoresession -n demo -w

NAME                 REPOSITORY   PHASE       DURATION   AGE
deployment-restore   gcs-repo     Succeeded   10s        35s
```

So, we can see from the output of the above command that the restore process succeeded.

### Verify Restored Data

In this section, we are going to verify whether the desired data has been restored successfully or not.

```bash
$ kubectl get pods -n demo -l app='stash-recovered'
NAME                               READY   STATUS    RESTARTS   AGE
stash-recovered-56547b7b57-scxl4   1/1     Running   0          16m
stash-recovered-56547b7b57-w4rf5   1/1     Running   0          16m
stash-recovered-56547b7b57-zxb2p   1/1     Running   0          16m
```

Verify that the backed up data has been restored in `/source/data` directory of the `stash-recovered` pods of the Deployment using the following commands,

```bash
$ kubectl exec -n demo stash-recovered-56547b7b57-scxl4 -- cat /source/data/data.txt
sample_data
```

## Cleaning Up

To clean up the Kubernetes resources created by this tutorial in `prod` and `staging` clusters, run the following commands:

**Cleanup `prod` Cluster**

Switch your cluster to `prod`,

```bash
$ kubectl config use-context kind-prod
Switched to context "kind-prod".
```

Clean up the Stash resources in the `prod` cluster,

```bash
kubectl delete -n demo deployments stash-demo
kubectl delete -n demo pvc stash-sample-data
kubectl delete -n demo backupconfiguration deployment-backup
kubectl delete -n demo repository gcs-repo
kubectl delete -n demo secret gcs-secret
```

**Cleanup `staging` Cluster**

Switch your cluster to `staging`,

```bash
$ kubectl config use-context kind-staging
Switched to context "kind-staging".
```

Clean up the Stash resources in the `staging` cluster,

```bash
kubectl delete -n demo deployments stash-recovered
kubectl delete -n demo pvc demo-pvc
kubectl delete -n demo restoresession deployment-restore
kubectl delete -n demo repository gcs-repo
kubectl delete -n demo secret gcs-secret
```