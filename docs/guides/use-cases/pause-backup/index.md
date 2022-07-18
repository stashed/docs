---
title: Pause Backup | Stash
description: An step by step guide on how to pause a scheduled backup in Stash.
menu:
  docs_{{ .version }}:
    identifier: use-cases-pause-backup
    name: Pause Backup
    parent: use-cases
    weight: 20
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: guides
---

# Pause Backup

Stash supports pausing backups without deleting respective `BackupConfiguration`. This guide will show you how to pause a scheduled backup in Stash.

## Before You Begin

- At first, you need to have a Kubernetes cluster. If you do not already have a cluster, you can create one by using [kind](https://kind.sigs.k8s.io/docs/user/quick-start/).

- Install `Stash` in your cluster following the steps [here](/docs/setup/README.md).

- You should be familiar with the following `Stash` concepts:
  - [BackupConfiguration](/docs/concepts/crds/backupconfiguration/index.md)
  - [BackupSession](/docs/concepts/crds/backupsession/index.md)
  - [Repository](/docs/concepts/crds/repository/index.md)

To keep everything isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```bash
$ kubectl create ns demo
namespace/demo created
```

> **Note:** YAML files used in this tutorial are stored in [docs/guides/use-cases/pause-backup/examples](https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/use-cases/pause-backup/examples) directory of [stashed/doc](https://github.com/stashed/doc) repository.

## Configure Backup

At first, we need to schedule a backup for a sample workload. Here, we are going to deploy a Deployment with a PVC. This Deployment will automatically generate some sample data into the PVC. Then, we are going to configure a scheduled backup for this Deployment.

**Deploy Deployment:**

Below are the YAMLs of the Deployment and PVC that we are going to create,

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: source-pvc
  namespace: demo
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
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
          claimName: source-pvc
```

The above Deployment will automatically create a `data.txt` file in `/source/data` directory and write some sample data in it.

Let's create the Deployment and PVC we have shown above.

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/use-cases/pause-backup/examples/deployment.yaml
persistentvolumeclaim/source-pvc created
deployment.apps/stash-demo created
```

Now, wait for the pods of the Deployment to go into the `Running` state.

```bash
kubectl get pod -n demo
NAME                          READY   STATUS    RESTARTS   AGE
stash-demo-69f9ffbbf7-bww24   1/1     Running   0          100s
stash-demo-69f9ffbbf7-r8wgh   1/1     Running   0          100s
stash-demo-69f9ffbbf7-rsj55   1/1     Running   0          100s
```

To verify that the sample data has been created in `/source/data` directory, use the following command:

```bash
$ kubectl exec -n demo stash-demo-69f9ffbbf7-bww24 -- cat /source/data/data.txt
sample_data
```

**Create Secret and Repository:**

We are going to store our backed up data into a [GCS bucket](https://cloud.google.com/storage/). We have to create a Secret and a Repository object with access credentials and backend information respectively.

> For GCS backend, if the bucket does not exist, Stash needs `Storage Object Admin` role permissions to create the bucket. For more details, please check the following [guide](/docs/guides/backends/gcs/index.md).

Let’s create a secret called ` gcs-secret` with access credentials to our desired GCS bucket,

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
      bucket: appscode-qa
      prefix: /sample/data
    storageSecretName: gcs-secret
```

Let's create the Repository we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/use-cases/pause-backup/examples/repository.yaml
repository.stash.appscode.com/gcs-repo created
```

Now, we are ready to backup our sample data into this backend.

**Create BackupConfiguration:**

We have to create a `BackupConfiguration` crd targeting the `stash-demo` Deployment that we have deployed earlier. Stash will inject a sidecar container into the target. It will also create a `CronJob` to take a periodic backup of `/source/data` directory of the target.

Below is the YAML of the `BackupConfiguration` crd that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: pause-backup
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

Let's create the `BackupConfiguration` crd we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/use-cases/pause-backup/examples/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/pause-backup created
```

**Verify Sidecar:**

If everything goes well, Stash will inject a sidecar container into the `stash-demo` Deployment to take backup of `/source/data` directory. Let’s check that the sidecar has been injected successfully,

```bash
$ kubectl get pod -n demo
NAME                          READY   STATUS    RESTARTS   AGE
stash-demo-7489fcb7f5-jctj4   2/2     Running   0          117s
stash-demo-7489fcb7f5-mfqps   2/2     Running   0          112s
stash-demo-7489fcb7f5-zp8c5   2/2     Running   0          115s
```

Look at the pod. It now has 2 containers. If you view the resource definition of this pod, you will see that there is a container named `stash` which is running `run-backup` command.

**Verify CronJob:**

It will also create a `CronJob` with the schedule specified in `spec.schedule` field of `BackupConfiguration` crd.

Verify that the `CronJob` has been created using the following command,

```bash
$ watch -n 1 kubectl get backupconfiguration -n demo
Every 3.0s: kubectl get backupconfiguration -n demo                      suaas-appscode: Thu Aug  1 17:08:08 2019

NAMESPACE   NAME           TASK   SCHEDULE      PAUSED   AGE
demo        pause-backup          */1 * * * *            27s
```

**Wait for BackupSession Succeeded:**

Wait for the next schedule for backup. Run the following command to watch `BackupSession` crd,

```bash
$ watch -n 1 kubectl get backupssession -n demo
Every 3.0s: kubectl get backupssession -n demo                      suaas-appscode: Thu Aug  1 17:43:57 2019

NAME                      INVOKER-TYPE          INVOKER-NAME   PHASE       AGE
pause-backup-1564659789   BackupConfiguration   pause-backup   Succeeded   49s
```

We can see from the above output that the backup session has succeeded. This indicates that the volumes of the Deployment have been backed up in the backend successfully.

## Pause Scheduled Backup

Now, we are going to pause the scheduled backup without deleting respective `BackupConfiguration`. In order to do that, we have to set `spec.paused: true` in the respective `BackupConfiguration` crd.

When we set `spec.paused: true`, the following things are going to happen:

- Respective CronJob will not be removed. However, it will skip creating any new BackupSession for next the schedules.
- Stash sidecar container which is responsible for taking backup will not be removed. So, your workload will not restart. However, it will skip taking backup even if a BackupSession is created to trigger an instant backup.

Let's patch the BackupConfiguration crd `pause-backup` and set `spec.paused: true`,

```bash
$ kubectl patch backupconfiguration -n demo pause-backup --type="merge" --patch='{"spec": {"paused": true}}'
backupconfiguration.stash.appscode.com/pause-backup patched
```

**Verify Scheduled Backup Get Skipped:**

Now, wait for the next backup schedule. This time, the CronJob will not create any new BackupSession. Instead, it will write an event to the BackupConfiguration crd that it has skipped creating BackupSession because the backup is paused.

Let's describe the `BackupConfiguration` to verify that the event has been created,

```bash
$ kubectl describe backupconfiguration -n demo pause-backup
Name:         pause-backup
Namespace:    demo
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"stash.appscode.com/v1beta1","kind":"BackupConfiguration","metadata":{"annotations":{},"name":"pause-backup","namespace":"de...
API Version:  stash.appscode.com/v1beta1
Kind:         BackupConfiguration
Metadata:
  Creation Timestamp:  2019-08-02T06:13:05Z
  Finalizers:
    stash.appscode.com
  Generation:        2
  Resource Version:  39756
  Self Link:         /apis/stash.appscode.com/v1beta1/namespaces/demo/backupconfigurations/pause-backup
  UID:               96ed2068-a1da-419b-bae3-478f1b876000
Spec:
  Paused:  true
  Repository:
    Name:  gcs-repo
  Retention Policy:
    Keep Last:  5
    Name:       keep-last-5
    Prune:      true
  Schedule:     */1 * * * *
  Target:
    Paths:
      /source/data
    Ref:
      API Version:  apps/v1
      Kind:         Deployment
      Name:         stash-demo
    Volume Mounts:
      Mount Path:  /source/data
      Name:        source-data
Events:
  Type    Reason          Age   From                       Message
  ----    ------          ----  ----                       -------
  Normal  Backup Skipped  29s   Backup Triggering CronJob  Skipping creating BackupSession. Reason: Backup Configuration demo/pause-backup is paused.
```

**Verify Instant Backup Get Skipped:**

If we try to trigger an instant backup by creating a `BackupSession` manually, it will be ignored. The sidecar container will write an event to the BackupSession describing why it has skipped taking the backup.

Below is the YAML of the `BackupSession` that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupSession
metadata:
  labels:
    stash.appscode.com/backup-configuration: pause-backup
  name: instant-backupsession
  namespace: demo
spec:
  invoker:
    apiGroup: stash.appscode.com
    kind: BackupConfiguration
    name: pause-backup
```

Let's create the `BackupSession` we have shown above.

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/use-cases/pause-backup/examples/backupsession.yaml
backupsession.stash.appscode.com/instant-backupsession created
```

Run the following command to watch the BackupSession phase,

```bash
$ watch -n 1 kubectl get backupsession -n demo instant-backupsession
Every 1.0s: kubectl get backupsession -n demo instant-backupsession  suaas-appscode: Fri Aug  2 11:56:24 2019

NAME                    INVOKER-TYPE          INVOKER-NAME   PHASE     AGE
instant-backupsession   BackupConfiguration   pause-backup   Skipped   3m22s
```

Notice the `PHASE` column. It is showing that the `BackupSession` has been skipped.

If you describe the `BackupSession` object you are going to see that there is an event explaining why it has been skipped.

```bash
$ kubectl describe backupsession -n demo instant-backupsession
Name:         instant-backupsession
Namespace:    demo
Labels:       stash.appscode.com/backup-configuration=pause-backup
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"stash.appscode.com/v1beta1","kind":"BackupSession","metadata":{"annotations":{},"labels":{"stash.appscode.com/backup-config...
API Version:  stash.appscode.com/v1beta1
Kind:         BackupSession
Metadata:
  Creation Timestamp:  2019-08-02T06:16:53Z
  Generation:          1
  Resource Version:    40010
  Self Link:           /apis/stash.appscode.com/v1beta1/namespaces/demo/backupsessions/instant-backupsession
  UID:                 6a44adab-e44e-4020-9c23-7545e3b3f13b
Spec:
  Invoker:
      API Group:  stash.appscode.com
      Kind:       BackupConfiguration
      Name:       pause-backup
Status:
  Phase:  Skipped
Events:
  Type     Reason                Age   From                      Message
  ----     ------                ----  ----                      -------
  Warning  BackupSessionSkipped  5s    BackupSession Controller  Backup Configuration is paused
  Warning  BackupSessionSkipped  5s    BackupSession Controller  Backup Configuration is paused
```

## Resume Backup

You can resume backup by setting `spec.paused: false` in BackupConfiguration crd. and applying the update or you can patch BackupConfiguration using,

```bash
$ kubectl patch backupconfiguration -n demo pause-backup --type="merge" --patch='{"spec": {"paused": false}}'
backupconfiguration.stash.appscode.com/pause-backup patched
```

## Cleaning Up

To clean up the Kubernetes resources created by this tutorial, run:

```bash
kubectl delete -n demo deployment stash-demo
kubectl delete -n demo backupconfiguration pause-backup
kubectl delete -n demo repository gce-repo
kubectl delete -n demo backupsession deployment-backupsession
kubectl delete -n demo secret gce-secret
kubectl delete -n demo pvc --all
```
