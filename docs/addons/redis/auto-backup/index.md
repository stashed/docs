---
title: Redis Auto-Backup | Stash
description: Backup Redis using Stash Auto-Backup
menu:
  docs_{{ .version }}:
    identifier: stash-redis-auto-backup
    name: Auto-Backup
    parent: stash-redis
    weight: 30
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: stash-addons
---

# Backup Redis using Stash Auto-Backup

Stash can be configured to automatically backup any Redis database in your cluster. Stash enables cluster administrators to deploy backup blueprints ahead of time so that the database owners can easily backup their database with just a few annotations.

In this tutorial, we are going to show how you can configure a backup blueprint for Redis databases in your cluster and backup them with few annotations.

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster.
- Install Stash Enterprise in your cluster following the steps [here](/docs/setup/install/enterprise.md).
- If you are not familiar with how Stash backup and restore Redis databases, please check the following guide [here](/docs/addons/redis/overview/index.md).
- If you are not familiar with how auto-backup works in Stash, please check the following guide [here](/docs/guides/auto-backup/overview.md).
- If you are not familiar with the available auto-backup options for databases in Stash, please check the following guide [here](/docs/guides/auto-backup/database.md).

You should be familiar with the following `Stash` concepts:

- [BackupBlueprint](/docs/concepts/crds/backupblueprint/index.md)
- [BackupConfiguration](/docs/concepts/crds/backupconfiguration/index.md)
- [BackupSession](/docs/concepts/crds/backupsession/index.md)
- [Repository](/docs/concepts/crds/repository/index.md)
- [Function](/docs/concepts/crds/function/index.md)
- [Task](/docs/concepts/crds/task/index.md)

In this tutorial, we are going to show backup of three different Redis databases on three different namespaces named `demo-1`, `demo-2`, and `demo-3`. Create the namespaces as below if you haven't done it already.

```bash
❯ kubectl create ns demo-1
namespace/demo-1 created

❯ kubectl create ns demo-2
namespace/demo-2 created

❯ kubectl create ns demo-3
namespace/demo-3 created
```

When you install Stash Enterprise, it installs the necessary addons to backup Redis. Verify that the Redis addons were installed properly using the following command.

```bash
❯ kubectl get tasks.stash.appscode.com | grep redis
redis-backup-6.2.5   62m
redis-backup-6.2.5   62m
```

We are going to use [bitnami/redis](https://artifacthub.io/packages/helm/bitnami/redis)  chart from [ArtifactHub](https://artifacthub.io/). Let's add the respective chart repository to our helm repo list.

```bash
# Add bitnami chart registry
$ helm repo add bitnami https://charts.bitnami.com/bitnami
# Update helm registries
$ helm repo update
```

## Prepare Backup Blueprint

To backup a Redis database using Stash, you have to create a `Secret` containing the backend credentials, a `Repository` containing the backend information, and a `BackupConfiguration` containing the schedule and target information. A `BackupBlueprint` allows you to specify a template for the `Repository` and the `BackupConfiguration`.

The `BackupBlueprint` is a non-namespaced CRD. So, once you have created a `BackupBlueprint`, you can use it to backup any Redis database of any namespace just by creating the storage `Secret` in that namespace and adding few annotations to the AppBinding containing the connection info to your database. Then, Stash will automatically create a `Repository` and a `BackupConfiguration` according to the template to backup the database.

Below is the `BackupBlueprint` object that we are going to use in this tutorial,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupBlueprint
metadata:
  name: redis-backup-template
spec:
  # ============== Blueprint for Repository ==========================
  backend:
    gcs:
      bucket: stash-testing
      prefix: redis-backup/${TARGET_NAMESPACE}/${TARGET_APP_RESOURCE}/${TARGET_NAME}
    storageSecretName: gcs-secret
  # ============== Blueprint for BackupConfiguration =================
  task:
    name: redis-backup-6.2.5
  schedule: "*/5 * * * *"
  retentionPolicy:
    name: 'keep-last-5'
    keepLast: 5
    prune: true
```

Here, we are using a GCS bucket as our backend. We are providing `gcs-secret` at the `storageSecretName` field. Hence, we have to create a secret named `gcs-secret` with the access credentials of our bucket in every namespace where we want to enable backup through this blueprint.

Notice the `prefix` field of `backend` section. We have used some variables in form of `${VARIABLE_NAME}`. Stash will automatically resolve those variables from the database information to make the backend prefix unique for each database instance.

Let's create the `BackupBlueprint` we have shown above,

```bash
❯ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/addons/redis/auto-backup/examples/backupblueprint.yaml
backupblueprint.stash.appscode.com/redis-backup-template created
```

Now, we are ready to backup our Redis databases using few annotations. You can check available auto-backup annotations for a databases from [here](/docs/guides/auto-backup/database.md#available-auto-backup-annotations-for-database).

## Auto-backup with default configurations

In this section, we are going to backup a Redis database of `demo-1` namespace. We are going to use the default configurations specified in the `BackupBlueprint`.

### Create Storage Secret

At first, let's create the `gcs-secret` in `demo-1` namespace with the access credentials to our GCS bucket.

```bash
❯ echo -n 'changeit' > RESTIC_PASSWORD
❯ echo -n '<your-project-id>' > GOOGLE_PROJECT_ID
❯ cat downloaded-sa-json.key > GOOGLE_SERVICE_ACCOUNT_JSON_KEY
❯ kubectl create secret generic -n demo-1 gcs-secret \
    --from-file=./RESTIC_PASSWORD \
    --from-file=./GOOGLE_PROJECT_ID \
    --from-file=./GOOGLE_SERVICE_ACCOUNT_JSON_KEY
secret/gcs-secret created
```

### Deploy Database

Let's deploy a Redis database named `sample-redis-1` in the `demo-1` namespace.

```bash
❯ helm install sample-redis-1 bitnami/redis -n demo-1
```

Now, let's insert some sample data into it.

```bash
❯ export PASSWORD=$(kubectl get secrets -n demo-1 sample-redis-1 -o jsonpath='{.data.\redis-password}' | base64 -d)
❯ kubectl exec -it -n demo-1 sample-redis-1-master-0 -- redis-cli -a $PASSWORD
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
127.0.0.1:6379> set key1 value1
OK
127.0.0.1:6379> get key1
"value1"
127.0.0.1:6379> exit
```

### Create AppBinding

Now, we have to create an AppBinding with the connection information of our database. Below, is the YAML of the AppBinding that we are going to create for our `sample-redis-1` database.

```yaml
apiVersion: appcatalog.appscode.com/v1alpha1
kind: AppBinding
metadata:
  name: sample-redis-1
  namespace: demo-1
  annotations:
    stash.appscode.com/backup-blueprint: redis-backup-template
spec:
  clientConfig:
    service:
      name: sample-redis-1-master
      path: /
      port: 6379
      scheme: http
  secret:
    name: sample-redis-1
  secretTransforms:
  - renameKey:
      from: redis-password
      to: password
  type: redis
  version: 6.2.5
```

Notice the `annotations` section. We are pointing to the `BackupBlueprint` that we have created earlier through `stash.appscode.com/backup-blueprint` annotation. Stash will watch this annotation and create a `Repository` and a `BackupConfiguration` according to the `BackupBlueprint`.

Let's create the above AppBinding,

```bash
❯ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/addons/redis/auto-backup/examples/sample-redis-1.yaml
appbinding.appcatalog.appscode.com/sample-redis-1 created
```

### Verify Auto-backup configured

In this section, we are going to verify whether Stash has created the respective `Repository` and `BackupConfiguration` for our Redis database or not.

#### Verify Repository

At first, let's verify whether Stash has created a `Repository` for our Redis or not.

```bash
❯ kubectl get repository -n demo-1
NAME                 INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
app-sample-redis-1                                                                22s
```

Now, let's check the YAML of the `Repository`.

```yaml
❯ kubectl get repository -n demo-1 app-sample-redis-1 -o yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: app-sample-redis-1
  namespace: demo-1
  ...
spec:
  backend:
    gcs:
      bucket: stash-testing
      prefix: redis-backup/demo-1/redis/sample-redis-1
    storageSecretName: gcs-secret
```

Here, you can see that Stash has resolved the variables in `prefix` field and substituted them with the equivalent information from this database.

#### Verify BackupConfiguration

If everything goes well, Stash should create a `BackupConfiguration` for our Redis in `demo-1` namespace and the phase of that `BackupConfiguration` should be `Ready`. Verify the `BackupConfiguration` crd by the following command,

```bash
❯ kubectl get backupconfiguration -n demo-1
NAME                 TASK                 SCHEDULE      PAUSED   PHASE   AGE
app-sample-redis-1   redis-backup-6.2.5   */5 * * * *            Ready   76s
```

Now, let's check the YAML of the `BackupConfiguration`.

```yaml
❯ kubectl get backupconfiguration -n demo-1 app-sample-redis-1 -o yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: app-sample-redis-1
  namespace: demo-1
  ...
spec:
  driver: Restic
  repository:
    name: app-sample-redis-1
  retentionPolicy:
    keepLast: 5
    name: keep-last-5
    prune: true
  runtimeSettings: {}
  schedule: '*/5 * * * *'
  target:
    ref:
      apiVersion: appcatalog.appscode.com/v1alpha1
      kind: AppBinding
      name: sample-redis-1
  task:
    name: redis-backup-6.2.5
  tempDir: {}
status:
  conditions:
  - lastTransitionTime: "2021-07-29T13:59:57Z"
    message: Repository demo-1/app-sample-redis-1 exist.
    reason: RepositoryAvailable
    status: "True"
    type: RepositoryFound
  - lastTransitionTime: "2021-07-29T13:59:57Z"
    message: Backend Secret demo-1/gcs-secret exist.
    reason: BackendSecretAvailable
    status: "True"
    type: BackendSecretFound
  - lastTransitionTime: "2021-07-29T13:59:57Z"
    message: Backup target appcatalog.appscode.com/v1alpha1 appbinding/sample-redis-1
      found.
    reason: TargetAvailable
    status: "True"
    type: BackupTargetFound
  - lastTransitionTime: "2021-07-29T13:59:57Z"
    message: Successfully created backup triggering CronJob.
    reason: CronJobCreationSucceeded
    status: "True"
    type: CronJobCreated
  observedGeneration: 1

```

Notice the `target` section. Stash has automatically added the Redis as the target of this `BackupConfiguration`.

#### Verify Backup

Now, let's wait for a backup run to complete. You can watch for `BackupSession` as below,

```bash
❯ kubectl get backupsession -n demo-1 -w
NAME                            INVOKER-TYPE          INVOKER-NAME         PHASE   DURATION   AGE
app-sample-redis-1-1627567808   BackupConfiguration   app-sample-redis-1                      0s
app-sample-redis-1-1627567808   BackupConfiguration   app-sample-redis-1   Running              22s
app-sample-redis-1-1627567808   BackupConfiguration   app-sample-redis-1   Succeeded   1m28.696008079s   88s
```

Once the backup has been completed successfully, you should see the backed up data has been stored in the bucket at the directory pointed by the `prefix` field of the `Repository`.

<figure align="center">
  <img alt="Backup data in GCS Bucket" src="/docs/addons/redis/auto-backup/images/sample-redis-1.png">
  <figcaption align="center">Fig: Backup data in GCS Bucket</figcaption>
</figure>

## Auto-backup with a custom schedule

In this section, we are going to backup a Redis database of `demo-2` namespace. This time, we are going to overwrite the default schedule used in the `BackupBlueprint`.

### Create Storage Secret

At first, let's create the `gcs-secret` in `demo-2` namespace with the access credentials to our GCS bucket.

```bash
❯ kubectl create secret generic -n demo-2 gcs-secret \
    --from-file=./RESTIC_PASSWORD \
    --from-file=./GOOGLE_PROJECT_ID \
    --from-file=./GOOGLE_SERVICE_ACCOUNT_JSON_KEY
secret/gcs-secret created
```

### Deploy Database

Let's deploy a Redis database named `sample-redis-2` in the `demo-2` namespace.

```bash
❯ helm install sample-redis-2 bitnami/redis -n demo-2
```

Now, let's insert some sample data into it.

```bash
❯ export PASSWORD=$(kubectl get secrets -n demo-2 sample-redis-2 -o jsonpath='{.data.\redis-password}' | base64 -d)
❯ kubectl exec -it -n demo-2 sample-redis-2-master-0 -- redis-cli -a $PASSWORD
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
127.0.0.1:6379> set key1 value1
OK
127.0.0.1:6379> get key1
"value1"
127.0.0.1:6379> exit
```

### Create AppBinding

Now, we have to create an AppBinding with the connection information of our database. Below, is the YAML of the AppBinding that we are going to create for our `sample-redis-2` database.

```yaml
apiVersion: appcatalog.appscode.com/v1alpha1
kind: AppBinding
metadata:
  name: sample-redis-2
  namespace: demo-2
  annotations:
    stash.appscode.com/backup-blueprint: redis-backup-template
    stash.appscode.com/schedule: "*/3 * * * *"
spec:
  clientConfig:
    service:
      name: sample-redis-2-master
      path: /
      port: 6379
      scheme: http
  secret:
    name: sample-redis-2
  secretTransforms:
  - renameKey:
      from: redis-password
      to: password
  type: redis
  version: 6.2.5
```

Notice the `annotations` section. This time, we have passed a schedule via `stash.appscode.com/schedule` annotation along with the `stash.appscode.com/backup-blueprint` annotation.

Let's create the above AppBinding,

```bash
❯ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/addons/redis/auto-backup/examples/sample-redis-2.yaml
appbinding.appcatalog.appscode.com/sample-redis-2 created
```

### Verify Auto-backup configured

Now, let's verify whether the auto-backup has been configured properly or not.

#### Verify Repository

At first, let's verify whether Stash has created a `Repository` for our Redis or not.

```bash
❯ kubectl get repository -n demo-2
NAME                 INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
app-sample-redis-2                                                                29s
```

Now, let's check the YAML of the `Repository`.

```yaml
❯ kubectl get repository -n demo-2 app-sample-redis-2  -o yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: app-sample-redis-2
  namespace: demo-2
  ...
spec:
  backend:
    gcs:
      bucket: stash-testing
      prefix: redis-backup/demo-2/redis/sample-redis-2
    storageSecretName: gcs-secret
```

Here, you can see that Stash has resolved the variables in `prefix` field and substituted them with the equivalent information from this new database.

#### Verify BackupConfiguration

If everything goes well, Stash should create a `BackupConfiguration` for our Redis in `demo-2` namespace and the phase of that `BackupConfiguration` should be `Ready`. Verify the `BackupConfiguration` crd by the following command,

```bash
❯ kubectl get backupconfiguration -n demo-2
NAME                 TASK                 SCHEDULE      PAUSED   PHASE   AGE
app-sample-redis-2   redis-backup-6.2.5   */3 * * * *            Ready   64s
```

Now, let's check the YAML of the `BackupConfiguration`.

```yaml
❯ kubectl get backupconfiguration -n demo-2 app-sample-redis-2 -o yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: app-sample-redis-2
  namespace: demo-2
  ...
spec:
  driver: Restic
  repository:
    name: app-sample-redis-2
  retentionPolicy:
    keepLast: 5
    name: keep-last-5
    prune: true
  runtimeSettings: {}
  schedule: '*/3 * * * *'
  target:
    ref:
      apiVersion: appcatalog.appscode.com/v1alpha1
      kind: AppBinding
      name: sample-redis-2
  task:
    name: redis-backup-6.2.5
  tempDir: {}
status:
  conditions:
  - lastTransitionTime: "2021-07-29T14:17:31Z"
    message: Repository demo-2/app-sample-redis-2 exist.
    reason: RepositoryAvailable
    status: "True"
    type: RepositoryFound
  - lastTransitionTime: "2021-07-29T14:17:31Z"
    message: Backend Secret demo-2/gcs-secret exist.
    reason: BackendSecretAvailable
    status: "True"
    type: BackendSecretFound
  - lastTransitionTime: "2021-07-29T14:17:31Z"
    message: Backup target appcatalog.appscode.com/v1alpha1 appbinding/sample-redis-2
      found.
    reason: TargetAvailable
    status: "True"
    type: BackupTargetFound
  - lastTransitionTime: "2021-07-29T14:17:31Z"
    message: Successfully created backup triggering CronJob.
    reason: CronJobCreationSucceeded
    status: "True"
    type: CronJobCreated
  observedGeneration: 1
```

Notice the `schedule` section. This time the `BackupConfiguration` has been created with the schedule we have provided via annotation.

Also, notice the `target` section. Stash has automatically added the new Redis as the target of this `BackupConfiguration`.

#### Verify Backup

Now, let's wait for a backup run to complete. You can watch for `BackupSession` as below,

```bash
❯ kubectl get backupsession -n demo-2 -w
NAME                            INVOKER-TYPE          INVOKER-NAME         PHASE     DURATION   AGE
app-sample-redis-2-1627568283   BackupConfiguration   app-sample-redis-2   Running              86s
app-sample-redis-2-1627568283   BackupConfiguration   app-sample-redis-2   Succeeded   1m33.522226054s   93s
```

Once the backup has been completed successfully, you should see that Stash has created a new directory as pointed by the `prefix` field of the new `Repository` and stored the backed up data there.

<figure align="center">
  <img alt="Backup data in GCS Bucket" src="/docs/addons/redis/auto-backup/images/sample-redis-2.png">
  <figcaption align="center">Fig: Backup data in GCS Bucket</figcaption>
</figure>

## Auto-backup with custom parameters

In this section, we are going to backup a Redis database of `demo-3` namespace. This time, we are going to pass some parameters for the Task through the annotations.

### Create Storage Secret

At first, let's create the `gcs-secret` in `demo-3` namespace with the access credentials to our GCS bucket.

```bash
❯ kubectl create secret generic -n demo-3 gcs-secret \
    --from-file=./RESTIC_PASSWORD \
    --from-file=./GOOGLE_PROJECT_ID \
    --from-file=./GOOGLE_SERVICE_ACCOUNT_JSON_KEY
secret/gcs-secret created
```

### Deploy Database

Let's deploy a Redis database named `sample-redis-3` in the `demo-3` namespace.

```bash
❯ helm install sample-redis-3 bitnami/redis -n demo-3
```

Now, let's insert some sample data into it.

```bash
❯ export PASSWORD=$(kubectl get secrets -n demo-3 sample-redis-3 -o jsonpath='{.data.\redis-password}' | base64 -d)
❯ kubectl exec -it -n demo-3 sample-redis-3-master-0 -- redis-cli -a $PASSWORD
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
127.0.0.1:6379> set key1 value1
OK
127.0.0.1:6379> get key1
"value1"
127.0.0.1:6379> exit
```

### Create AppBinding

Now, we have to create an AppBinding with the connection information of our database. Below, is the YAML of the AppBinding that we are going to create for our `sample-redis-3` database.

```yaml
apiVersion: appcatalog.appscode.com/v1alpha1
kind: AppBinding
metadata:
  name: sample-redis-3
  namespace: demo-3
  annotations:
    stash.appscode.com/backup-blueprint: redis-backup-template
    params.stash.appscode.com/args: -db 0
spec:
  clientConfig:
    service:
      name: sample-redis-3-master
      path: /
      port: 6379
      scheme: http
  secret:
    name: sample-redis-3
  secretTransforms:
  - renameKey:
      from: redis-password
      to: password
  type: redis
  version: 6.2.5
```

Notice the `annotations` section. This time, we have passed an argument via `params.stash.appscode.com/args` annotation along with the `stash.appscode.com/backup-blueprint` annotation.

Let's create the above AppBinding,

```bash
❯ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/addons/redis/auto-backup/examples/sample-redis-3.yaml
appbinding.appcatalog.appscode.com/sample-redis-3 created
```

### Verify Auto-backup configured

Now, let's verify whether the auto-backup resources has been created or not.

#### Verify Repository

At first, let's verify whether Stash has created a `Repository` for our Redis or not.

```bash
❯ kubectl get repository -n demo-3
NAME                 INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
app-sample-redis-3                                                                29s
```

Now, let's check the YAML of the `Repository`.

```yaml
❯ kubectl get repository -n demo-3 app-sample-redis-3 -o yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: app-sample-redis-3
  namespace: demo-3
  ...
spec:
  backend:
    gcs:
      bucket: stash-testing
      prefix: redis-backup/demo-3/redis/sample-redis-3
    storageSecretName: gcs-secret
```

Here, you can see that Stash has resolved the variables in `prefix` field and substituted them with the equivalent information from this new database.

#### Verify BackupConfiguration

If everything goes well, Stash should create a `BackupConfiguration` for our Redis in `demo-3` namespace and the phase of that `BackupConfiguration` should be `Ready`. Verify the `BackupConfiguration` crd by the following command,

```bash
❯ kubectl get backupconfiguration -n demo-3
NAME                 TASK                 SCHEDULE      PAUSED   PHASE   AGE
app-sample-redis-3   redis-backup-6.2.5   */5 * * * *            Ready   62s
```

Now, let's check the YAML of the `BackupConfiguration`.

```yaml
❯ kubectl get backupconfiguration -n demo-3 app-sample-redis-3 -o yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: app-sample-redis-3
  namespace: demo-3
  ...
spec:
  driver: Restic
  repository:
    name: app-sample-redis-3
  retentionPolicy:
    keepLast: 5
    name: keep-last-5
    prune: true
  runtimeSettings: {}
  schedule: '*/5 * * * *'
  target:
    ref:
      apiVersion: appcatalog.appscode.com/v1alpha1
      kind: AppBinding
      name: sample-redis-3
  task:
    name: redis-backup-6.2.5
    params:
    - name: args
      value: -db 0
  tempDir: {}
status:
  conditions:
  - lastTransitionTime: "2021-07-29T14:23:58Z"
    message: Repository demo-3/app-sample-redis-3 exist.
    reason: RepositoryAvailable
    status: "True"
    type: RepositoryFound
  - lastTransitionTime: "2021-07-29T14:23:58Z"
    message: Backend Secret demo-3/gcs-secret exist.
    reason: BackendSecretAvailable
    status: "True"
    type: BackendSecretFound
  - lastTransitionTime: "2021-07-29T14:23:58Z"
    message: Backup target appcatalog.appscode.com/v1alpha1 appbinding/sample-redis-3
      found.
    reason: TargetAvailable
    status: "True"
    type: BackupTargetFound
  - lastTransitionTime: "2021-07-29T14:23:58Z"
    message: Successfully created backup triggering CronJob.
    reason: CronJobCreationSucceeded
    status: "True"
    type: CronJobCreated
  observedGeneration: 1
```

Notice the `task` section. The `args` parameter that we had passed via annotations has been added to the `params` section.

Also, notice the `target` section. Stash has automatically added the new Redis as the target of this `BackupConfiguration`.

#### Verify Backup

Now, let's wait for a backup run to complete. You can watch for `BackupSession` as below,

```bash
❯ kubectl get backupsession -n demo-3 -w
NAME                            INVOKER-TYPE          INVOKER-NAME         PHASE     DURATION            AGE
app-sample-redis-3-1627568709   BackupConfiguration   app-sample-redis-3   Running                       20s
app-sample-redis-3-1627568709   BackupConfiguration   app-sample-redis-3   Succeeded   1m43.931692282s   103s
```

Once the backup has been completed successfully, you should see that Stash has created a new directory as pointed by the `prefix` field of the new `Repository` and stored the backed up data there.

<figure align="center">
  <img alt="Backup data in GCS Bucket" src="/docs/addons/redis/auto-backup/images/sample-redis-3.png">
  <figcaption align="center">Fig: Backup data in GCS Bucket</figcaption>
</figure>

## Cleanup

To cleanup the resources crated by this tutorial, run the following commands,

```bash
# cleanup sample-redis-1 resources
❯ helm uninstall sample-redis-1 -n demo-1
❯ kubectl delete appbinding sample-redis-1 -n demo-1
❯ kubectl delete repository -n demo-1 --all

# cleanup sample-redis-2 resources
❯ helm uninstall sample-redis-2 -n demo-2
❯ kubectl delete appbinding sample-redis-2 -n demo-2
❯ kubectl delete repository -n demo-2 --all

# cleanup sample-redis-3 resources
❯ helm uninstall sample-redis-3 -n demo-3
❯ kubectl delete appbinding sample-redis-3 -n demo-3
❯ kubectl delete repository -n demo-3 --all

# cleanup BackupBlueprint
❯ kubectl delete backupblueprint redis-backup-template
```
