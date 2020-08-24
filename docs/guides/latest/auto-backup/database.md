---
title: Auto Backup Databases | Stash
description: An step by step guide on how to configure automatic backup for Databases.
menu:
  docs_{{ .version }}:
    identifier: auto-backup-database
    name: Auto Backup for Databases
    parent: auto-backup
    weight: 40
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: guides
---

# Auto Backup for Database

This tutorial will show you how to configure automatic backup for PostgreSQL database using Stash. Here, we are going to backup two different PostgreSQL databases of two different version using a common blueprint.

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [kind](https://kind.sigs.k8s.io/docs/user/quick-start/).
- Install Stash in your cluster following the steps [here](/docs/setup/install.md).
- Install PostgreSQL addon for Stash following the steps [here](/docs/addons/postgres/setup/install.md).
- Install [KubeDB](https://kubedb.com) in your cluster following the steps [here](https://kubedb.com/docs/latest/setup/install/). This step is optional. You can deploy your database using any method you want. We are using KubeDB because KubeDB simplifies many of the difficult or tedious management tasks of running a production grade databases on private and public clouds.
- If you are not familiar with how Stash backup and restore PostgreSQL databases, please check the following guide [here](/docs/addons/postgres/overview.md).

You should be familiar with the following `Stash` concepts:

- [BackupBlueprint](/docs/concepts/crds/backupblueprint.md)
- [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md)
- [BackupSession](/docs/concepts/crds/backupsession.md)
- [Repository](/docs/concepts/crds/repository.md)
- [Function](/docs/concepts/crds/function.md)
- [Task](/docs/concepts/crds/task.md)

To keep everything isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$ kubectl create ns demo
namespace/demo created
```

> **Note:** YAML files used in this tutorial are stored in [docs/examples/guides/latest/auto-backup/database](/docs/examples/guides/latest/auto-backup/database) directory of [stashed/docs](https://github.com/stashed/docs) repository.

## Prepare Backup Blueprint

We are going to use [GCS Backend](/docs/guides/latest/backends/gcs.md) to store the backed up data. You can use any supported backend you prefer. You just have to configure Storage Secret and `spec.backend` section of `BackupBlueprint` to match with your backend. To learn which backends are supported by Stash and how to configure them, please visit [here](/docs/guides/latest/backends/overview.md).

> For GCS backend, if the bucket does not exist, Stash needs `Storage Object Admin` role permissions to create the bucket. For more details, please check the following [guide](/docs/guides/latest/backends/gcs.md).

**Create Storage Secret:**

At first, let's create a Storage Secret for the GCS backend,

```console
$ echo -n 'changeit' > RESTIC_PASSWORD
$ echo -n '<your-project-id>' > GOOGLE_PROJECT_ID
$ mv downloaded-sa-json.key GOOGLE_SERVICE_ACCOUNT_JSON_KEY
$ kubectl create secret generic -n demo gcs-secret \
    --from-file=./RESTIC_PASSWORD \
    --from-file=./GOOGLE_PROJECT_ID \
    --from-file=./GOOGLE_SERVICE_ACCOUNT_JSON_KEY
secret/gcs-secret created
```

**Create BackupBlueprint:**

Now, we have to create a `BackupBlueprint` crd with a blueprint for `Repository` and `BackupConfiguration` object.

Below is the YAML of the `BackupBlueprint` object that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupBlueprint
metadata:
  name: postgres-backup-blueprint
spec:
  # ============== Blueprint for Repository ==========================
  backend:
    gcs:
      bucket: appscode-qa
      prefix: stash-backup/${TARGET_NAMESPACE}/${TARGET_APP_RESOURCE}/${TARGET_NAME}
    storageSecretName: gcs-secret
  # ============== Blueprint for BackupConfiguration =================
  task:
    name: postgres-backup-${TARGET_APP_VERSION}
  schedule: "*/5 * * * *"
  retentionPolicy:
    name: 'keep-last-5'
    keepLast: 5
    prune: true
```

Here,

- `spec.task.name` specifies the `Task` crd name that will be used to backup the targeted database. We have used a variable `${TARGET_APP_VERSION}` as task name suffix. This variable will be substituted by the respective database version. This allows to backup multiple database versions with a common blueprint.

Note that we have used some variables (format: `${<variable name>}`) in `spec.backend.gcs.prefix` field. Stash will substitute these variables with values from the respective target. To learn which variables you can use in the `prefix` field, please visit [here](/docs/concepts/crds/backupblueprint.md#repository-blueprint).

Let's create the `BackupBlueprint` that we have shown above,

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/auto-backup/database/backupblueprint.yaml
backupblueprint.stash.appscode.com/postgres-backup-blueprint created
```

Now, automatic backup is configured for PostgreSQL database. We just have to add a annotation to the `AppBinding` of the targeted database.

> Note: `BackupBlueprint` is a non-namespaced crd. So, you can use a `BackupBlueprint` to backup targets in multiple namespaces. However, Storage Secret is a namespaced object. So, you have to manually create the secret in each namespace where you have a target for backup. Please give us your feedback on how to improve the ux of this aspect of Stash on [GitHub](https://github.com/stashed/stash/issues/842).

**Available Auto-Backup Annotations for Database:**

You have to add the auto-backup annotations to the `AppBinding` CR of the targeted database. If you are using KubeDB, you can add these annotations to the respective database CR. KubeDB will pass the annotations into the respective `AppBinding`.

The following auto-backup annotations are available for databases:

- **BackupBlueprint Name:** You have to specify the `BackupBlueprint` name that holds the template for `Repository` and `BackupConfiguration` in the following annotation:

```yaml
stash.appscode.com/backup-blueprint: <BackupBlueprint name>
```

- **Schedule:** You can specify a schedule to backup this target through this annotation. If you don't specify this annotation, schedule from the `BackupBlueprint` will be used.

```yaml
 stash.appscode.com/schedule: <Cron Expression>
```

- **Task Parameters:** You can also pass some parameters to the respective backup Task through annotations. Use following format to pass parameters via annotations:

```yaml
params.stash.appscode.com/key1: value1
params.stash.appscode.com/key2: value2,value3
params.stash.appscode.com/key3: ab=123,bc=234
```

## Prepare Databases

Now, we are going to deploy two sample PostgreSQL databases of two different versions using KubeDB. We are going to backup these two databases using auto-backup.

**Deploy First PostgreSQL Sample:**

Below is the YAML of the first `Postgres` crd,

```yaml
apiVersion: kubedb.com/v1alpha1
kind: Postgres
metadata:
  name: sample-postgres-1
  namespace: demo
spec:
  version: "11.2"
  storageType: Durable
  storage:
    storageClassName: "standard"
    accessModes:
    - ReadWriteOnce
    resources:
      requests:
        storage: 1Gi
  terminationPolicy: Delete
```

Let's create the `Postgres` we have shown above,

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/auto-backup/database/sample-postgres-1.yaml
postgres.kubedb.com/sample-postgres-1 created
```

KubeDB will deploy a PostgreSQL database according to the above specification and it will create the necessary secrets and services to access the database. It will also create an `AppBinding` crd that holds the necessary information to connect with the database.

Verify that an `AppBinding` has been created for this PostgreSQL sample,

```console
$ kubectl get appbinding -n demo
NAME                AGE
sample-postgres-1   47s
```

If you view the YAML of this `AppBinding`, you will see it holds service and secret information. Stash uses this information to connect with the database.

```console
$ kubectl get appbinding -n demo sample-postgres-1 -o yaml
```

```yaml
apiVersion: appcatalog.appscode.com/v1alpha1
kind: AppBinding
metadata:
  name: sample-postgres-1
  namespace: demo
  ...
spec:
  clientConfig:
    service:
      name: sample-postgres-1
      path: /
      port: 5432
      query: sslmode=disable
      scheme: postgresql
  secret:
    name: sample-postgres-1-auth
  secretTransforms:
  - renameKey:
      from: POSTGRES_USER
      to: username
  - renameKey:
      from: POSTGRES_PASSWORD
      to: password
  type: kubedb.com/postgres
  version: "11.2"
```

> If you have deployed the database without KubeDB, you have to create the AppBinding crd manually.

**Deploy Second PostgreSQL Sample:**

Below is the YAML of second `Postgres` object,

```yaml
apiVersion: kubedb.com/v1alpha1
kind: Postgres
metadata:
  name: sample-postgres-2
  namespace: demo
spec:
  version: "10.6-v2"
  storageType: Durable
  storage:
    storageClassName: "standard"
    accessModes:
    - ReadWriteOnce
    resources:
      requests:
        storage: 1Gi
  terminationPolicy: Delete
```

Let's create the `Postgres` we have shown above,

```console
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/latest/auto-backup/database/sample-postgres-2.yaml
postgres.kubedb.com/sample-postgres-2 created
```

Verify that an `AppBinding` has been created for this PostgreSQL database,

```console
$ kubectl get appbinding -n demo
NAME                AGE
sample-postgres-1   2m49s
sample-postgres-2   10s
```

Here, we can see `AppBinding` `sample-postgres-2` has been created for our second PostgreSQL sample.

## Backup

Now, we are going to add auto-backup specific annotation to the `AppBinding` of our desired database. Stash watches for `AppBinding` crd. Once it finds an `AppBinding` with auto-backup annotation, it will create a `Repository` and a `BackupConfiguration` crd according to respective `BackupBlueprint`. Then, rest of the backup process will proceed as normal database backup as described [here](/docs/guides/latest/addons/overview.md).

### Backup First PostgreSQL Sample

Let's backup our first PostgreSQL sample using auto-backup.

**Add Annotations:**

At first, add the auto-backup specific annotation to the AppBinding `sample-postgres-1`,

```console
$ kubectl annotate appbinding sample-postgres-1 -n demo --overwrite \
  stash.appscode.com/backup-blueprint=postgres-backup-blueprint
```

Verify that the annotation has been added successfully,

```console
$ kubectl get appbinding -n demo sample-postgres-1 -o yaml
```

```yaml
apiVersion: appcatalog.appscode.com/v1alpha1
kind: AppBinding
metadata:
  annotations:
    stash.appscode.com/backup-blueprint: postgres-backup-blueprint
  name: sample-postgres-1
  namespace: demo
  ...
spec:
  clientConfig:
    service:
      name: sample-postgres-1
      path: /
      port: 5432
      query: sslmode=disable
      scheme: postgresql
  secret:
    name: sample-postgres-1-auth
  secretTransforms:
  - renameKey:
      from: POSTGRES_USER
      to: username
  - renameKey:
      from: POSTGRES_PASSWORD
      to: password
  type: kubedb.com/postgres
  version: "11.2"
```

Now, Stash will create a `Repository` crd and a `BackupConfiguration` crd according to the blueprint.

**Verify Repository:**

Verify that the `Repository` has been created successfully by the following command,

```console
$ kubectl get repository -n demo
NAME                         INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
postgres-sample-postgres-1                                                                2m23s
```

If we view the YAML of this `Repository`, we are going to see that the variables `${TARGET_NAMESPACE}`, `${TARGET_APP_RESOURCE}` and `${TARGET_NAME}` has been replaced by `demo`, `postgres` and `sample-postgres-1` respectively.

```console
$ kubectl get repository -n demo postgres-sample-postgres-1 -o yaml
```

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: Repository
metadata:
  creationTimestamp: "2019-08-01T13:54:48Z"
  finalizers:
  - stash
  generation: 1
  name: postgres-sample-postgres-1
  namespace: demo
  resourceVersion: "50171"
  selfLink: /apis/stash.appscode.com/v1beta1/namespaces/demo/repositories/postgres-sample-postgres-1
  uid: ed49dde4-b463-11e9-a6a0-080027aded7e
spec:
  backend:
    gcs:
      bucket: appscode-qa
      prefix: stash-backup/demo/postgres/sample-postgres-1
    storageSecretName: gcs-secret
```

**Verify BackupConfiguration:**

Verify that the `BackupConfiguration` crd has been created by the following command,

```console
$ kubectl get backupconfiguration -n demo
NAME                         TASK                   SCHEDULE      PAUSED   AGE
postgres-sample-postgres-1   postgres-backup-11.2   */5 * * * *            3m39s
```

Notice the `TASK` field. It denoting that this backup will be performed using `postgres-backup-11.2` task. We had specified `postgres-backup-${TARGET_APP_VERSION}` as task name in the `BackupBlueprint`. Here, the variable `${TARGET_APP_VERSION}` has been substituted by the database version.

Let's check the YAML of this `BackupConfiguration`,

```console
$ kubectl get backupconfiguration -n demo postgres-sample-postgres-1 -o yaml
```

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  creationTimestamp: "2019-08-01T13:54:48Z"
  finalizers:
  - stash.appscode.com
  generation: 1
  name: postgres-sample-postgres-1
  namespace: demo
  ownerReferences:
  - apiVersion: v1
    blockOwnerDeletion: false
    kind: AppBinding
    name: sample-postgres-1
    uid: a799156e-b463-11e9-a6a0-080027aded7e
  resourceVersion: "50170"
  selfLink: /apis/stash.appscode.com/v1beta1/namespaces/demo/backupconfigurations/postgres-sample-postgres-1
  uid: ed4bd257-b463-11e9-a6a0-080027aded7e
spec:
  repository:
    name: postgres-sample-postgres-1
  retentionPolicy:
    keepLast: 5
    name: keep-last-5
    prune: true
  runtimeSettings: {}
  schedule: '*/5 * * * *'
  target:
    ref:
      apiVersion: v1
      kind: AppBinding
      name: sample-postgres-1
  task:
    name: postgres-backup-11.2
  tempDir: {}
```

Notice that the `spec.target.ref` is pointing to the AppBinding `sample-postgres-1` that we have just annotated with auto-backup annotation.

**Wait for BackupSession:**

Now, wait for the next backup schedule. Run the following command to watch `BackupSession` crd:

```console
$ watch -n 1 kubectl get backupsession -n demo -l=stash.appscode.com/backup-configuration=postgres-sample-postgres-1

Every 1.0s: kubectl get backupsession -n demo -l=stash.appscode.com/backup-configuration=postgres-sample-postgres-1  workstation: Thu Aug  1 20:35:43 2019

NAME                                    INVOKER-TYPE          INVOKER-NAME                 PHASE       AGE
postgres-sample-postgres-1-1564670101   BackupConfiguration   postgres-sample-postgres-1   Succeeded   42s
```

> Note: Backup CronJob creates `BackupSession` crd with the following label `stash.appscode.com/backup-configuration=<BackupConfiguration crd name>`. We can use this label to watch only the `BackupSession` of our desired `BackupConfiguration`.

**Verify Backup:**

When backup session is completed, Stash will update the respective `Repository` to reflect the latest state of backed up data.

Run the following command to check if a snapshot has been sent to the backend,

```console
$ kubectl get repository -n demo postgres-sample-postgres-1
NAME                         INTEGRITY   SIZE        SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
postgres-sample-postgres-1   true        1.324 KiB   1                73s                      6m7s
```

If we navigate to `stash-backup/demo/postgres/sample-postgres-1` directory of our GCS bucket, we are going to see that the snapshot has been stored there.

<figure align="center">
  <img alt="Backup data of PostgreSQL 'sample-postgres-1' in GCS backend" src="/docs/images/guides/latest/auto-backup/database/sample_postgres_1.png">
  <figcaption align="center">Fig: Backup data of PostgreSQL "sample-postgres-1" in GCS backend</figcaption>
</figure>

### Backup Second Sample PostgreSQL

Now, lets backup our second PostgreSQL sample using the same `BackupBlueprint` we have used to backup the first PostgreSQL sample.

**Add Annotations:**

Add the auto backup specific annotation to AppBinding `sample-postgres-2`,

```console
$ kubectl annotate appbinding sample-postgres-2 -n demo --overwrite \
  stash.appscode.com/backup-blueprint=postgres-backup-blueprint
```

**Verify Repository:**

Verify that the `Repository` has been created successfully by the following command,

```console
$ kubectl get repository -n demo
NAME                         INTEGRITY   SIZE        SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
postgres-sample-postgres-1   true        1.324 KiB   1                2m3s                     6m57s
postgres-sample-postgres-2                                                                     15s
```

Here, Repository `postgres-sample-postgres-2` has been created for the second PostgreSQL sample.

If we view the YAML of this `Repository`, we are going to see that the variables `${TARGET_NAMESPACE}`, `${TARGET_APP_RESOURCE}` and `${TARGET_NAME}` has been replaced by `demo`, `postgres` and `sample-postgres-2` respectively.

```console
$ kubectl get repository -n demo postgres-sample-postgres-2 -o yaml
```

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: Repository
metadata:
  creationTimestamp: "2019-08-01T14:37:22Z"
  finalizers:
  - stash
  generation: 1
  name: postgres-sample-postgres-2
  namespace: demo
  resourceVersion: "56103"
  selfLink: /apis/stash.appscode.com/v1beta1/namespaces/demo/repositories/postgres-sample-postgres-2
  uid: df58523c-b469-11e9-a6a0-080027aded7e
spec:
  backend:
    gcs:
      bucket: appscode-qa
      prefix: stash-backup/demo/postgres/sample-postgres-2
    storageSecretName: gcs-secret
```

**Verify BackupConfiguration:**

Verify that the `BackupConfiguration` crd has been created by the following command,

```console
$ kubectl get backupconfiguration -n demo
NAME                         TASK                   SCHEDULE      PAUSED   AGE
postgres-sample-postgres-1   postgres-backup-11.2   */5 * * * *            7m52s
postgres-sample-postgres-2   postgres-backup-10.6   */5 * * * *            70s
```

Again, notice the `TASK` field. This time, `${TARGET_APP_VERSION}` has been replaced with `10.6` which is the database version of our second sample.

**Wait for BackupSession:**

Now, wait for the next backup schedule. Run the following command to watch `BackupSession` crd:

```console
$ watch -n 1 kubectl get backupsession -n demo -l=stash.appscode.com/backup-configuration=postgres-sample-postgres-2
Every 1.0s: kubectl get backupsession -n demo -l=stash.appscode.com/backup-configuration=postgres-sample-postgres-2  workstation: Thu Aug  1 20:55:40 2019

NAME                                    INVOKER-TYPE          INVOKER-NAME                 PHASE       AGE
postgres-sample-postgres-2-1564671303   BackupConfiguration   postgres-sample-postgres-2   Succeeded   37s
```

**Verify Backup:**

Run the following command to check if a snapshot has been sent to the backend,

```console
$ kubectl get repository -n demo postgres-sample-postgres-2
NAME                         INTEGRITY   SIZE        SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
postgres-sample-postgres-2   true        1.324 KiB   1                52s                      19m
```

If we navigate to `stash-backup/demo/postgres/sample-postgres-2` directory of our GCS bucket, we are going to see that the snapshot has been stored there.

<figure align="center">
  <img alt="Backup data of PostgreSQL 'sample-postgres-2' in GCS backend" src="/docs/images/guides/latest/auto-backup/database/sample_postgres_2.png">
  <figcaption align="center">Fig: Backup data of PostgreSQL "sample-postgres-2" in GCS backend</figcaption>
</figure>

## Cleanup

To cleanup the Kubernetes resources created by this tutorial, run:

```console
kubectl delete -n demo pg/sample-postgres-1
kubectl delete -n demo pg/sample-postgres-2

kubectl delete -n demo repository/postgres-sample-postgres-1
kubectl delete -n demo repository/postgres-sample-postgres-2

kubectl delete -n demo backupblueprint/postgres-backup-blueprint
```

If you would like to uninstall Stash operator, please follow the steps [here](/docs/setup/uninstall.md).
