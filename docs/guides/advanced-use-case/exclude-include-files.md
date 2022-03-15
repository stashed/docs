---
title: Exclude/Include Files | Stash
description: A guide on how to exclude/include subset of files during backup/restore process.
menu:
docs_{{ .version }}:
identifier: advance-use-case-cross-namespace-backup
name: Cross-Namespace Backup and Restore
parent: advance-use-case
weight: 10
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: guides
---

# Exclude/Include Subset of Files During Backup/Restore Process.
This guide will show you how to exclude/include subset of files during backup/restore process.

## Before You Begin
- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [kind](https://kind.sigs.k8s.io/docs/user/quick-start/).

- Install `Stash` in your cluster following the steps [here](/docs/setup/README.md).

- You should be familiar with the following `Stash` concepts:
  - [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md)
  - [BackupSession](/docs/concepts/crds/backupsession.md)
  - [RestoreSession](/docs/concepts/crds/restoresession.md)
  - [Repository](/docs/concepts/crds/repository.md)

To keep everything isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```bash
$ kubectl create ns demo
namespace/demo created
```
> **Note:** YAML files used in this tutorial are stored in [docs/examples/guides/advanced-use-case/exclude-include-files](/docs/examples/guides/advanced-use-case/exclude-include-files) directory of [stashed/docs](https://github.com/stashed/docs) repository.

## Configure Backup

Here, we are going to configure backup for a subset of files using `exclude`/`include`. At first, we are going to deploy a Deployment with a PVC and generate some sample data in it. Then, we are going to configure the backup for the Deployment using Stash. 

#### Deploy Deployment:

Below is the YAML of the Deployment and PVC that we are going to create,

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: source-data
  namespace: demo
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 2Gi
---
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
      - args: ["echo sample-text-1 > /source/data/data1.txt && 
                echo sample-text-2 > /source/data/data2.txt && 
                echo not-important > /source/data/not-important.txt && 
                echo sample-html > /source/data/web/index.html &&
                sleep 3000"]
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
          claimName: source-data
```

The above Deployment will automatically create a `data1.txt`, `data2.txt`, `index.html`, file in `/source/data` directory and write some sample data in it.

Let’s create the Deployment and PVC we have shown above.

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/advanced-use-case/exclude-include-files/deployment.yaml
persistentvolumeclaim/source-data created
deployment.apps/stash-demo created
```

Now, wait for the pod of the Deployment to go into `Running` state.

```bash
$ kubectl get pod -n demo
NAME                          READY   STATUS    RESTARTS   AGE
stash-demo-859d96f6bd-fxr7l   1/1     Running   0          81s
```

Verify that the files have been created in `/source/data` directory using the following command,

```bash
$ kubectl exec -n demo stash-demo-859d96f6bd-fxr7l -- ls /source/data
data1.txt
data2.txt
index.html
```

#### Create Secret and Repository:

We are going to store our backed up data into a GCS bucket. We have to create a Secret and a Repository object with access credentials and backend information respectively.

> For GCS backend, if the bucket does not exist, Stash needs `Storage Object Admin` role permissions to create the bucket. For more details, please check the following [guide](/docs/guides/backends/gcs.md).

Let’s create a secret called `gcs-secret` with access credentials of our desired GCS backend,

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
      prefix: /stash/filter/deployment
    storageSecretName: gcs-secret
```

Let’s create the `Repository` object that we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/advanced-use-case/exclude-include-files/repository.yaml
repository.stash.appscode.com/gcs-repo created
```

Now, we are ready to backup our sample data into this backend.

#### Create BackupConfiguration 

We have to create a `BackupConfiguration` crd targeting the `stash-demo` Deployment that we have deployed earlier. Then, Stash will inject a sidecar container into the target. It will also create a `CronJob` to take periodic backup of `/source/data` directory of the target.

You can exclude a particular file during a backup. Below is the YAML of the `BackupConfiguration` crd that we are going to create,

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
    exclude:
    - /source/data/not-important.txt # exclude only one file
    - /source/data/*.html # exclude all file that has a particular extension
    - /source/data/tmp/* # exclude a directory
  retentionPolicy:
    name: 'keep-last-5'
    keepLast: 5
    prune: true
```
The above configuration will backup everything inside the `/source/data` directory except `source/data/not-important.txt`, all `html` files, and the directory `source/data/tmp`.
Let’s create the `BackupConfiguration` object that we have shown above,
```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/advanced-use-case/exclude-include-files/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/deployment-backup created
```
If everything goes well, the phase of the `BackupConfiguration` should be `Ready`. The `Ready` phase indicates that the backup setup is successful. 

##### Trigger Backup
```bash
`kubectl stash trigger -n demo
```
##### Wait for BackupSession to Succeed:

Run the following command to watch `BackupSession` phase,

```bash
$ watch kubectl get -n demo backupsession -n demo
NAME                       INVOKER-TYPE          INVOKER-NAME        PHASE       AGE
deployment-backupsession   BackupConfiguration   deployment-backup   Succeeded   21s
```
##### Verify Backup
Download the `latest` snapshot of the Repository `gcs-repo` to verify our backup data.

```bash
kubectl stash download gcs-repo -n demo --destination=/tmp/ --snapshots="latest"
```
Navigate to the directory `/tmp` to verify the backup data.

##### Excluding all the files that has a particular extension:
You can exclude all the files that has a particular extension during a backup. Below is the YAML of the `BackupConfiguration` object to backup everything in the `source/data` directory except the `html` files.
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
    exclude:
    -  /source/data/*.html 
  retentionPolicy:
    name: 'keep-last-5'
    keepLast: 5
    prune: true
```
##### Excluding a directory:
You can also exclude a particular directory during a backup. Below is the YAML of the `BackupConfiguration` object to backup everything in the `/source/data*` directory except the directory `source/data/tmp`.

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
    exclude:
    - /source/data/tmp/* 
  retentionPolicy:
    name: 'keep-last-5'
    keepLast: 5
    prune: true
```


