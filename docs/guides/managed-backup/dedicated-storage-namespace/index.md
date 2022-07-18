---
title: Dedicated Storage Namespace | Stash
description: A guide on how to use backup and restore by keeping the storage resources in a separate namespace in Stash.
menu:
  docs_{{ .version }}:
    identifier: managed-backup-dedicated-storage-namespace
    name: Dedicated Storage Namespace
    parent: managed-backup
    weight: 20
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: guides
---

# Using Dedicated Namespace for Storage

This guide will show you how to take backup and restore by keeping the storage resources (Repository and backend Secret) in a dedicated namespace using Stash.

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [kind](https://kind.sigs.k8s.io/docs/user/quick-start/).

- Install `Stash` in your cluster following the steps [here](/docs/setup/README.md).

- You should be familiar with the following `Stash` concepts:
  - [BackupConfiguration](/docs/concepts/crds/backupconfiguration/index.md)
  - [BackupSession](/docs/concepts/crds/backupsession/index.md)
  - [RestoreSession](/docs/concepts/crds/restoresession/index.md)
  - [Repository](/docs/concepts/crds/repository/index.md)

We are going to take a backup from the `prod` namespace and restore it to the `staging` namespace. We are going to keep our Repository and the backend Secret in the `storage` namespace.

Let's create the above-mentioned namespaces,

```bash
$ kubectl create ns prod
namespace/prod created

$ kubectl create ns staging
namespace/staging created

$ kubectl create ns storage
namespace/storage created
```

>**Note:** YAML files used in this tutorial can be found [here](https://github.com/stashed/docs/guides/managed-backup/dedicated-storage-namespace/examples).

## Backup from `prod` Namespace

This section will demonstrate taking backup of the volumes of a StatefulSet from the `prod` namespace using Stash.

### Deploy Workload

Let's deploy a StatefulSet in the `prod` namespace at the beginning. This StatefulSet will automatically generate sample data in the `/source/data` directory.

Here is the YAML of the StatefulSet that we are going to create,

```yaml
apiVersion: v1
kind: Service
metadata:
  name: headless
  namespace: prod
spec:
  ports:
  - name: http
    port: 80
    targetPort: 0
  selector:
    app: stash-demo
  clusterIP: None
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: stash-demo
  namespace: prod
  labels:
    app: stash-demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: stash-demo
  serviceName: headless
  template:
    metadata:
      labels:
        app: stash-demo
    spec:
      containers:
      - name: busybox
        image: busybox
        command: ["/bin/sh", "-c","echo $(POD_NAME) > /source/data/data.txt && sleep 3000"]
        env:
        - name:  POD_NAME
          valueFrom:
            fieldRef:
              fieldPath:  metadata.name
        volumeMounts:
        - name: source-data
          mountPath: "/source/data"
        imagePullPolicy: IfNotPresent
  volumeClaimTemplates:
  - metadata:
      name: source-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "standard"
      resources:
        requests:
          storage: 1Gi
```

Let's create the StatefulSet we have shown above.

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/managed-backup/dedicated-storage-namespace/examples/statefulset.yaml
service/headless created
statefulset.apps/stash-demo created
```

Now, wait for the pods of the StatefulSet to go into the `Running` state. You can verify the `Status` of the pods by executing the following command,

```bash
$ kubectl get pod -n prod
NAME           READY   STATUS    RESTARTS   AGE
stash-demo-0   1/1     Running   0          42s
stash-demo-1   1/1     Running   0          40s
stash-demo-2   1/1     Running   0          36s
```

Verify that the sample data has been generated in `/source/data` directory for `stash-demo-0` , `stash-demo-1` and `stash-demo-2` pod respectively using the following commands,

```bash
$ kubectl exec -n prod stash-demo-0 -- cat /source/data/data.txt
stash-demo-0
$ kubectl exec -n prod stash-demo-1 -- cat /source/data/data.txt
stash-demo-1
$ kubectl exec -n prod stash-demo-2 -- cat /source/data/data.txt
stash-demo-2
```

### Prepare Backend

We are going to store our backed-up data into a GCS bucket. We have to create a Secret with the necessary credentials and a Repository object to use this backend. 

If you want to use a different backend, please read the doc [here](/docs/guides/backends/overview/index.md).

> For the GCS backend, if the bucket does not exist, Stash needs `Storage Object Admin` role permissions to create the bucket. For more details, please check the following [guide](/docs/guides/backends/gcs.md).

**Create Secret:**

Let's create a secret called `gcs-secret` in `storage` namespace with access credentials to our desired GCS bucket,

```bash
$ echo -n 'changeit' > RESTIC_PASSWORD
$ echo -n '<your-project-id>' > GOOGLE_PROJECT_ID
$ cat /path/to/downloaded-sa-key.json > GOOGLE_SERVICE_ACCOUNT_JSON_KEY
$ kubectl create secret generic -n storage gcs-secret \
    --from-file=./RESTIC_PASSWORD \
    --from-file=./GOOGLE_PROJECT_ID \
    --from-file=./GOOGLE_SERVICE_ACCOUNT_JSON_KEY
secret/gcs-secret created
```

**Create Repository:**

Now, create a Repository using the above Secret in the `storage` namespace. Below is the YAML of Repository object we are going to create, 

```yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: gcs-repo
  namespace: storage
spec:
  backend:
    gcs:
      bucket: stash-testing
      prefix: /cross-namespace/data/sample-statefulset
    storageSecretName: gcs-secret
  usagePolicy:
    allowedNamespaces:
      from: All
```

Notice the `spec.usagePolicy` section. Here, we are allowing all namespaces to refer to this repository. You can restrict this capability to a particular namespace or a group of namespaces. For more details, please follow the guide from [here](/docs/concepts/crds/repository/index.md).

Let's create the Repository we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/managed-backup/dedicated-storage-namespace/examples/repository.yaml
repository.stash.appscode.com/gcs-repo created
```

Now, we are ready to backup our sample data into this backend.

### Configure Backup

We are going to create a `BackupConfiguration` object in the `prod` namespace targeting the `stash-demo` StatefulSet. This `BackupConfiguration` will refer to the `gcs-repo` repository of the `storage` namespace. Stash will inject a sidecar container into the target. It will also create a `CronJob` to take periodic backup of the `/source/data` directory of the target.

**Create BackupConfiguration:**

Below is the YAML of the `BackupConfiguration` object that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: ss-backup
  namespace: prod
spec:
  repository:
    name: gcs-repo
    namespace: storage
  schedule: "*/5 * * * *"
  target:
    ref:
      apiVersion: apps/v1
      kind: StatefulSet
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
- `spec.repository.namespace` refers to the namespace of the Repository object.
- `spec.schedule` is a cron expression that indicates BackupSession will be created at 5 minutes intervals.
- `spec.target.ref` refers to the `stash-demo` StatefulSet.
- `spec.target.volumeMounts` specifies a list of volumes and their mountPath that contain the target paths.
- `spec.target.paths` specifies list of file paths to backup.

Let's create the `BackupConfiguration` crd we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/managed-backup/dedicated-storage-namespace/examples/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/ss-backup created
```

**Verify BackupConfiguration Ready:**

If everything goes well, the phase of the BackupConfiguration should be `Ready`. Let's check the BackupConfiguration Phase,

```bash
❯ kubectl get backupconfiguration -n prod
NAME        TASK   SCHEDULE      PAUSED   PHASE   AGE
ss-backup          */5 * * * *            Ready   13s
```

**Verify CronJob:**

Stash will also create a `CronJob` with the schedule specified in the `spec.schedule` field of the `BackupConfiguration` object.

Verify that the `CronJob` has been created using the following command,

```bash
$ kubectl get cronjob -n  prod
NAME                      SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
stash-trigger-ss-backup   */5 * * * *   False     0        4m55s           3m14s
```

### Verify Backup

The `stash-trigger-ss-backup` CronJob will trigger a backup on each scheduled slot by creating a `BackupSession` object. The sidecar container watches for the `BackupSession` object. When it finds one, it will take backup immediately.

Wait for the next schedule for the backup. Run the following command to watch the `BackupSession` object,

```bash
$ kubectl get backupsession -n prod -w

NAME                   INVOKER-TYPE          INVOKER-NAME    PHASE        DURATION   AGE
ss-backup-1644562803   BackupConfiguration   ss-backup       Running                 18s
ss-backup-1644562803   BackupConfiguration   ss-backup       Running                 34s
ss-backup-1644562803   BackupConfiguration   ss-backup       Succeeded    1m21s      81s
```

We can see from the above output that the backup session has succeeded.

## Restore into `staging` Namespace

This section will demonstrate restoring the backed-up volumes into the `staging` namespace using Stash.

**Stop Taking Backup of the Old StatefulSet:**

At first, let's stop taking any further backup of the old StatefulSet. We are going to pause the `BackupConfiguration` that we created earlier. You can learn more how to pause a scheduled backup [here](/docs/guides/use-cases/pause-backup.md)

Let's pause the `ss-backup` BackupConfiguration,

```bash
$ kubectl patch backupconfiguration -n prod ss-backup --type="merge" --patch='{"spec": {"paused": true}}'
backupconfiguration.stash.appscode.com/ss-backup patched
```

You can also use the Stash kubectl plugin to pause the backup like the following,

```bash
$ kubectl stash pause backup --backupconfig=ss-backup -n prod
BackupConfiguration demo/deployment-backup has been paused successfully.
```

Verify that the BackupConfiguration has been paused,

```bash
$ kubectl get backupconfiguration -n prod
NAME        TASK   SCHEDULE      PAUSED   PHASE   AGE
ss-backup          */5 * * * *   true     Ready   53m
```

Notice the `PAUSED` column. Value `true` indicates that the BackupConfiguration has been paused.

### Deploy Restore Workload

We are going to create a new StatefulSet named `stash-recovered` and restore the backed-up data inside it.

Below is the YAML of the StatefulSet that we are going to create,

```yaml
apiVersion: v1
kind: Service
metadata:
  name: re-headless
  namespace: staging
spec:
  ports:
  - name: http
    port: 80
    targetPort: 0
  selector:
    app: stash-recovered
  clusterIP: None
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: stash-recovered
  namespace: staging
  labels:
    app: stash-recovered
spec:
  replicas: 3
  selector:
    matchLabels:
      app: stash-recovered
  serviceName: re-headless
  template:
    metadata:
      labels:
        app: stash-recovered
    spec:
      containers:
      - name: busybox
        image: busybox
        command:
        - sleep
        - '3600'
        volumeMounts:
        - name: source-data
          mountPath: "/source/data"
        imagePullPolicy: IfNotPresent
  volumeClaimTemplates:
  - metadata:
      name: source-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "standard"
      resources:
        requests:
          storage: 1Gi
```

Let's create the StatefulSet we have shown above.

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/managed-backup/dedicated-storage-namespace/examples/statefulset_recovered.yaml
service/re-headless created
statefulset.apps/stash-recovered created
```

### Configure Restore

Now, we need to create a `RestoreSession` object targeting the `stash-recovered` StatefulSet to restore the backed-up data inside it. Similar to the BackupConfiguration, we will use the Repository of `storage` namespace here.

**Create RestoreSession:**

Below is the YAML of the `RestoreSesion` object that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: ss-restore
  namespace: staging
spec:
  repository:
    name: gcs-repo
    namespace: storage
  target:
    ref:
      apiVersion: apps/v1
      kind: StatefulSet
      name: stash-recovered
    volumeMounts:
    - name:  source-data
      mountPath:  /source/data
    rules:
    - paths:
      - /source/data
```

Here,

- `spec.repository.name` specifies the name of the Repository.
- `spec.repository.namespace` refers to the namespace of the Repository.
- `spec.target.ref` refers to the target workload where the recovered data will be stored.
- `spec.target.volumeMounts` specifies a list of volumes and their mountPath where the data will be restored.
  - `mountPath` must be the same mountPath as the original volume because Stash stores the absolute path of the backed-up files. If you use a different mountPath for the restored volume the backed up files will not be restored into your desired volume.

Let's create the `RestoreSession` object we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/managed-backup/dedicated-storage-namespace/examples/restoresession.yaml
restoresession.stash.appscode.com/ss-restore created
```

Now, wait for the RestoreSession phase to go into the `Succeeded` state.

**Wait for RestoreSession to Succeeded:**

Run the following command to watch the RestoreSession phase,

```bash
$ kubectl get restoresession -n staging -w

NAME         REPOSITORY   PHASE       DURATION   AGE
ss-restore   gcs-repo     Succeeded   3m6s       4m
```

So, we can see from the output of the above command that the restore process succeeded.

### Verify Restored Data

In this section, we are going to verify that the desired data has been restored successfully.

Get the pods of the `stash-recovered` StatefulSet,

```bash
$ kubectl get pod -n staging
NAME                READY   STATUS    RESTARTS   AGE
stash-recovered-0   1/1     Running   0          10m
stash-recovered-1   1/1     Running   0          11m
stash-recovered-2   1/1     Running   0          12m
```

Verify that the backed-up data has been restored in `/source/data` directory of the `stash-recovered` pods of a StatefulSet using the following commands,

```bash
$ kubectl exec -n staging stash-recovered-0 -- cat /source/data/data.txt
stash-demo-0
$ kubectl exec -n staging stash-recovered-1 -- cat /source/data/data.txt
stash-demo-1
$ kubectl exec -n staging stash-recovered-2 -- cat /source/data/data.txt
stash-demo-2
```

## Cleaning Up

To clean up the resources created by this tutorial, run:

```bash
kubectl delete -n prod statefulset stash-demo
kubectl delete -n prod backupconfiguration ss-backup
kubectl delete -n prod pvc --all
kubectl delete -n staging statefulset stash-recovered
kubectl delete -n staging restoresession ss-restore
kubectl delete -n staging pvc --all
kubectl delete -n storage repository gcs-repo
kubectl delete -n storage secret gcs-secret
```