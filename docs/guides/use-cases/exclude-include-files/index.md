---
title: Exclude/Include Files | Stash
description: A guide on how to filter files during backup/restore process.
menu:
  docs_{{ .version }}:
    identifier: use-cases-exclude-include-files
    name: Exclude/Include Files
    parent: use-cases
    weight: 12
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: guides
---

# Filtering Files During Backup/Restore

This guide will show you how to exclude/include subset of files during backup/restore process.

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [kind](https://kind.sigs.k8s.io/docs/user/quick-start/).
- Install `Stash` in your cluster following the steps [here](/docs/setup/README.md).
- Install Stash `kubectl` plugin following the steps [here](https://stash.run/docs/latest/setup/install/kubectl-plugin/).
- You should be familiar with the following `Stash` concepts:
  - [BackupConfiguration](/docs/concepts/crds/backupconfiguration/index.md)
  - [BackupSession](/docs/concepts/crds/backupsession/index.md)
  - [RestoreSession](/docs/concepts/crds/restoresession/index.md)
  - [Repository](/docs/concepts/crds/repository/index.md)

To keep everything isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```bash
❯ kubectl create ns demo
namespace/demo created
```

## Prepare Workload

At first, we are going to create a `PVC` then we are going to create a `Deployment` that will use this `PVC`.

### Create PVC

Below is the YAML of the sample PVC that we are going to create,

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
```

Let’s create the PVC we have shown above,

```bash
❯ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/use-cases/exclude-include-files/examples/pvc-source.yaml
persistentvolumeclaim/source-data created
```

### Create Deployment

Now, we are going to deploy a Deployment that uses the above PVC. Below is the YAML of the Deployment that we are going to create,

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

Let’s create the `Deployment` we have shown above,

```bash
❯ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/use-cases/exclude-include-files/examples/deployment-source.yaml
deployment.apps/stash-demo created
```

Now, wait for the pod of the Deployment to go into `Running` state.

```bash
❯ kubectl get pod -n demo
NAME                          READY   STATUS    RESTARTS   AGE
stash-demo-67576d874-2tj9d    1/1     Running   0          81s
```

### Insert Data

```bash
# create sample data
❯ kubectl exec -n demo -it  stash-demo-67576d874-2tj9d -- /bin/sh -c "touch /source/data/data-1.txt"

❯ kubectl exec -n demo -it  stash-demo-67576d874-2tj9d -- /bin/sh -c "touch /source/data/data-2.txt"

❯ kubectl exec -n demo -it  stash-demo-67576d874-2tj9d -- /bin/sh -c "touch /source/data/not-important.txt"

❯ kubectl exec -n demo -it  stash-demo-67576d874-2tj9d -- /bin/sh -c "touch /source/data/index.html"

❯ kubectl exec -n demo -it  stash-demo-67576d874-2tj9d -- /bin/sh -c "touch /source/data/resp.json"

❯ kubectl exec -n demo -it  stash-demo-67576d874-2tj9d -- /bin/sh -c "mkdir /source/data/tmp"

❯ kubectl exec -n demo -it  stash-demo-67576d874-2tj9d -- /bin/sh -c "touch /source/data/tmp/tmp.txt"
```

## Prepare Backend

We are going to store our backed up data into a GCS bucket. We have to create a Secret with necessary credentials and a Repository crd to use this backend. If you want to use a different backend, please read the respective backend configuration doc from [here](https://stash.run/docs/latest/guides/backends/overview/).

> For GCS backend, if the bucket does not exist, Stash needs Storage Object Admin role permissions to create the bucket. For more details, please check the following guide.

### Create Secret

Let’s create a secret called `gcs-secret` with access credentials to our desired GCS bucket,

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

## Full Backup

Stash automatically takes backup of all the data in the specified path. We are going to create a BackupConfiguration crd targeting the stash-demo Deployment that we have deployed earlier. Then, Stash will inject a sidecar container into the target. It will also create a CronJob to take periodic backup of `/source/data` directory of the target.

We are going to use the following `Repository` to backup our data,

```yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: gcs-repo-full
  namespace: demo
spec:
  backend:
    gcs:
      bucket: stash-testing
      prefix: /demo/stash-demo/full
    storageSecretName: gcs-secret
```

Let’s create the `Repository`,

```bash
❯ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/use-cases/exclude-include-files/examples/repository-full.yaml
repository.stash.appscode.com/gcs-repo-full created
```

Bellow is the yaml of the `BackupConfiguration` we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: deployment-backup-full
  namespace: demo
spec:
  repository:
    name: gcs-repo-full
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

The above `BackupConfiguration` will backup everything inside the `/source/data` directory.

Let’s create the `BackupConfiguration` object that we have shown above, 

```bash
❯ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/use-cases/exclude-include-files/examples/backupconfiguration-full.yaml
backupconfiguration.stash.appscode.com/deployment-backup-full created
```

If everything goes well, the phase of the `BackupConfiguration` should be `Ready`. The `Ready` phase indicates that the backup setup is successful. 

### Trigger a Backup

Lets trigger a backup using Stash `kubectl` plugin,

```bash
❯ kubectl stash trigger -n demo deployment-backup-full
```

### Wait for BackupSession to Succeed

Run the following command to watch `BackupSession` phase,

```bash
❯ watch kubectl get -n demo backupsession -n demo
NAME                                INVOKER-TYPE          INVOKER-NAME        PHASE       AGE
deployment-backup-full-1647347700   BackupConfiguration   deployment-backup   Succeeded   21s
```

### Verify Backup

Lets download the `latest` snapshot of the `Repository` in `/tmp/full-backup` directory using Stash `kubectl` plugin,

```bash
❯ mkdir /tmp/full-backup
❯ kubectl stash download gcs-repo-full -n demo --destination=/tmp/full-backup --snapshots="latest"
```

List the files in `/tmp/full-backup/latest/source/data` directory to verify the backup data.

```bash
❯ ls -R /tmp/full-backup/latest/source/data

/tmp/full-backup/latest/source/data:
data-1.txt  data-2.txt  index.html  not-important.txt  resp.json  tmp/

/tmp/full-backup/latest/source/data/tmp:
tmp.txt
```

## Filtering During Backup

In this section, we are going to show how to filter files during a backup.

### Exclude Subset of Files

Here, we are going show how to exclude particular files during a backup. We can exclude a subset of files during backup using the `spec.target.exclude` section in the `BackupConfiguration`. 

We are going to use the following `Repository` to backup our data,

```yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: gcs-repo-exclude
  namespace: demo
spec:
  backend:
    gcs:
      bucket: stash-testing
      prefix: /demo/stash-demo/exclude
    storageSecretName: gcs-secret

```

Let’s create the `Repository` we have shown above,

```bash
❯ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/use-cases/exclude-include-files/examples/repository-exclude.yaml
repository.stash.appscode.com/gcs-repo-exclude created
```


Below is the YAML of the `BackupConfiguration` crd that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: deployment-backup-exclude
  namespace: demo
spec:
  repository:
    name: gcs-repo-exclude
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
    - /source/data/*.html # exclude the files with .html extension
    - /source/data/tmp/* # exclude a directory
  retentionPolicy:
    name: 'keep-last-5'
    keepLast: 5
    prune: true

```

The above BackupConfiguration will backup everything inside the `/source/data` directory except the file `source/data/not-important.txt`, all the `html` files, and the directory `source/data/tmp`.

 Let’s create the BackupConfiguration object that we have shown above,

```bash
❯ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/use-cases/exclude-include-files/examples/backupconfiguration-exclude.yaml
backupconfiguration.stash.appscode.com/deployment-backup-exclude created
```

If everything goes well, the phase of the `BackupConfiguration` should be `Ready`. The `Ready` phase indicates that the backup setup is successful. 

#### Trigger a Backup

Lets trigger a backup using Stash `kubectl` plugin,

```bash
kubectl stash trigger -n demo deployment-backup-exclude
```

#### Wait for BackupSession to Succeed

Run the following command to watch `BackupSession` phase,

```bash
$ watch kubectl get -n demo backupsession -n demo
NAME                                   INVOKER-TYPE          INVOKER-NAME        PHASE       AGE
deployment-backup-exclude-1647347700   BackupConfiguration   deployment-backup   Succeeded   21s
```

### Verify Backup

Lets download the `latest` snapshot of the `Repository` in `/tmp/partial-backup` directory using Stash `kubectl` plugin,

```bash
❯ mkdir /tmp/partial-backup
❯ kubectl stash download gcs-repo-full -n demo --destination=/tmp/partial-backup --snapshots="latest"
```

List the files in `/tmp/partial-backup/latest/source/data` directory to verify the backup data.

```bash
❯ ls -R /tmp/partial-backup/latest/source/data

/tmp/partial-backup/latest/source/data:
data-1.txt  data-2.txt  resp.json  tmp/

/tmp/partial-backup/latest/source/data/tmp:
```

## Filtering During Restore

In this section, we are going to show how to filter files during a backup. At first, we are going to deploy a new Deployment with a PVC . Then, we are going to restore the backed up data using Stash.  

### Prepare Workload

Below is the YAML of the `PVC` that we are going to create,

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: recovered-data
  namespace: demo
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 1Gi

```

Let’s create the PVC we have shown above,

```bash
❯ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/use-cases/exclude-include-files/examples/pvc-recovered.yaml
persistentvolumeclaim/recovered-data created
```

Below is the YAML of the `Deployment` that we are going to create,

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: stash-recovered
  name: stash-recovered
  namespace: demo
spec:
  replicas: 1
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
          claimName: recovered-data

```

Let’s create the `Deployment` we have shown above,

```bash
❯ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/use-cases/exclude-include-files/examples/deployment-recovered.yaml
deployment.apps/stash-recovered created
```

Now, wait for the pod of the Deployment to go into `Running` state.

```bash
❯ kubectl get pod -n demo
NAME                               READY   STATUS    RESTARTS   AGE
stash-recovered-67576d874-2tj9d    1/1     Running   0          81s
```

### Exclude Subset of Files

Here, we are going show how to exclude particular files during a restore. We can exclude a subset of files during a restore using the `spec.target.rules` section in the `RestoreSession`. 

Below is the YAML of the `RestoreSession` crd that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: deployment-restore-exclude
  namespace: demo
spec:
  repository:
    name: gcs-repo-full
  target:
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
      exclude:
      - /source/data/not-important.txt # don't restore this file
      - /source/data/*.html # don't restore the files with .html extension
      - /source/data/tmp/* # don't restore this directory

```

The above `RestoreSession` will restore everything inside the `/source/data` directory except `source/data/not-important.txt`, all the `html` files, and the directory `source/data/tmp`.

Let’s create the `RestoreSession` object that we have shown above,

```bash
❯ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/use-cases/exclude-include-files/examples/restoresession-exclude.yaml
restoresession.stash.appscode.com/deployment-restore-exclude created
```

Verify that the files have been restored in `/source/data` directory using the following command,

```bash
❯ kubectl exec -n demo stash-recovered-67576d874-2tj9d -- ls -R /source/data

/source/data:
data-1.txt
data-2.txt
resp.json
tmp/

/source/data/tmp:
```

### Restore Subset of Files

Previously we have restored the backed up data excluding specific files or directory. You can also restore only the selected files or directory during  a restore. 

Below is the YAML of the `RestoreSession` crd that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: deployment-restore-include
  namespace: demo
spec:
  repository:
    name: gcs-repo-full
  target:
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
      include:
      - /source/data/data1.txt # restore this file
      - /source/data/*.json # restore the files with .json extension
      - /source/data/tmp/* # restore this directory
```

Let’s create the `RestoreSession` object that we have shown above,

```bash
❯ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/use-cases/exclude-include-files/examples/restoresession-include.yaml
restoresession.stash.appscode.com/deployment-restore-include created
```

Verify that the files have been restored in `/source/data` directory using the following command,

```bash
❯ kubectl exec -n demo stash-recovered-7dd74d9ff7-h9t7x -- ls -R /source/data
/source/data:
resp.json
tmp

/source/data/tmp:
tmp.txt
```

## Cleaning Up

```bash
❯ kubectl delete -n demo deployment stash-demo
❯ kubectl delete -n demo deployment stash-recovered
❯ kubectl delete -n demo backupconfiguration deployment-backup-full
❯ kubectl delete -n demo backupconfiguration deployment-backup-exclude
❯ kubectl delete -n demo restoresession deployment-restore-include
❯ kubectl delete -n demo restoresession deployment-restore-exclude
❯ kubectl delete -n demo repository gcs-repo-full
❯ kubectl delete -n demo repository gcs-repo-exclude
❯ kubectl delete -n demo secret gcs-secret
❯ kubectl delete -n demo pvc --all
```
