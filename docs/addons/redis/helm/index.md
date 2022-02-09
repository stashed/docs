---
title: Logical Backup & Restore Redis | Stash
description: Take logical backup of Redis database using Stash
menu:
  docs_{{ .version }}:
    identifier: stash-redis-helm
    name: Helm managed Redis
    parent: stash-redis
    weight: 20
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: stash-addons
---

# Take a logical backup of the Redis database using Stash

Stash `{{< param "info.version" >}}` supports backup and restoration of Redis databases. This guide will show you how you can take a logical backup of your Redis databases and restore them using Stash.

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster.
- Install Stash Enterprise in your cluster following the steps [here](/docs/setup/install/enterprise.md).
- If you are not familiar with how Stash backup and restore Redis databases, please check the following guide [here](/docs/addons/redis/overview/index.md).

You have to be familiar with following custom resources:

- [AppBinding](/docs/concepts/crds/appbinding.md)
- [Function](/docs/concepts/crds/function.md)
- [Task](/docs/concepts/crds/task.md)
- [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md)
- [BackupSession](/docs/concepts/crds/backupsession.md)
- [RestoreSession](/docs/concepts/crds/restoresession.md)

To keep things isolated, we are going to use a separate namespace called `demo` throughout this tutorial. Create `demo` namespace if you haven't created already.

```bash
$ kubectl create ns demo
namespace/demo created
```

> Note: YAML files used in this tutorial are stored [here](https://github.com/stashed/docs/tree/{{< param "info.version" >}}/docs/addons/redis/helm/examples).

## Prepare Redis

In this section, we are going to deploy a Redis database. Then, we are going to insert some sample data into it.

### Deploy Redis

At first, let's deploy a Redis database. Here, we are going to use [bitnami/redis](https://artifacthub.io/packages/helm/bitnami/redis)  chart from [ArtifactHub](https://artifacthub.io/).

Let's deploy a Redis database named `sample-redis` using Helm as below,

```bash
# Add bitnami chart registry
$ helm repo add bitnami https://charts.bitnami.com/bitnami
# Update helm registries
$ helm repo update
# Install bitnami/redis chart into demo namespace
$ helm install sample-redis bitnami/redis -n demo
```

This chart will create the necessary StatefulSet, Secret, Service etc. for the database. You can easily view all the resources created by chart using [ketall](https://github.com/corneliusweig/ketall) `kubectl` plugin as below,

```bash
❯ kubectl get-all -n demo -l app.kubernetes.io/instance=sample-redis
NAME                                                        NAMESPACE  AGE
configmap/sample-redis-configuration                        demo       117s  
configmap/sample-redis-health                               demo       117s  
configmap/sample-redis-scripts                              demo       117s  
endpoints/sample-redis-headless                             demo       117s  
endpoints/sample-redis-master                               demo       117s  
endpoints/sample-redis-replicas                             demo       117s  
persistentvolumeclaim/redis-data-sample-redis-master-0      demo       117s  
persistentvolumeclaim/redis-data-sample-redis-replicas-0    demo       117s  
persistentvolumeclaim/redis-data-sample-redis-replicas-1    demo       79s   
persistentvolumeclaim/redis-data-sample-redis-replicas-2    demo       51s   
pod/sample-redis-master-0                                   demo       117s  
pod/sample-redis-replicas-0                                 demo       117s  
pod/sample-redis-replicas-1                                 demo       79s   
pod/sample-redis-replicas-2                                 demo       51s   
secret/sample-redis                                         demo       117s  
serviceaccount/sample-redis                                 demo       117s  
service/sample-redis-headless                               demo       117s  
service/sample-redis-master                                 demo       117s  
service/sample-redis-replicas                               demo       117s  
controllerrevision.apps/sample-redis-master-755dd8b64d      demo       117s  
controllerrevision.apps/sample-redis-replicas-7b8c7694bf    demo       117s  
statefulset.apps/sample-redis-master                        demo       117s  
statefulset.apps/sample-redis-replicas                      demo       117s  
endpointslice.discovery.k8s.io/sample-redis-headless-6bvt2  demo       117s  
endpointslice.discovery.k8s.io/sample-redis-master-78wcv    demo       117s  
endpointslice.discovery.k8s.io/sample-redis-replicas-vhc7z  demo       117s 
```

Now, wait for the database pod `sample-redis-master-0` to go into `Running` state,

```bash
❯ kubectl get pod -n demo sample-redis-master-0
NAME                    READY   STATUS    RESTARTS   AGE
sample-redis-master-0   1/1     Running   0          2m57s
```

Once the database pod is in `Running` state, verify that the database is ready to accept the connections.

```bash
❯ kubectl logs -n demo sample-redis-master-0
1:C 28 Jul 2021 13:03:28.191 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1:C 28 Jul 2021 13:03:28.191 # Redis version=6.2.5, bits=64, commit=00000000, modified=0, pid=1, just started
1:C 28 Jul 2021 13:03:28.191 # Configuration loaded
1:M 28 Jul 2021 13:03:28.192 * monotonic clock: POSIX clock_gettime
1:M 28 Jul 2021 13:03:28.192 * Running mode=standalone, port=6379.
1:M 28 Jul 2021 13:03:28.192 # Server initialized
1:M 28 Jul 2021 13:03:28.193 * Ready to accept connections
```

From the above log, we can see the database is ready to accept connections.

### Insert Sample Data

Now, we are going to exec into the database pod and create some sample data. The helm chart has created a secret with access credentials. Let's find out the credentials from the Secret,

```yaml
❯ kubectl get secret -n demo sample-redis -o yaml
apiVersion: v1
data:
  redis-password: WTFZTENrZmNpcw==
kind: Secret
metadata:
  annotations:
    meta.helm.sh/release-name: sample-redis
    meta.helm.sh/release-namespace: demo
  creationTimestamp: "2021-07-28T13:03:23Z"
  labels:
    app.kubernetes.io/instance: sample-redis
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: redis
    helm.sh/chart: redis-14.8.6
  name: sample-redis
  namespace: demo
  resourceVersion: "530037"
  uid: a48ce23a-105d-4d92-9067-c80623cbe269
type: Opaque

```

Here, we are going to use `redis-password` to authenticate and insert the sample data.

At first, let's export the password as environment variables to make further commands re-usable.

```bash
export PASSWORD=$(kubectl get secrets -n demo sample-redis -o jsonpath='{.data.\redis-password}' | base64 -d)
```

Now, let's exec into the database pod and insert some sample data,

```bash
❯ kubectl exec -it -n demo sample-redis-master-0 -- redis-cli -a $PASSWORD
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
# insert some key value pairs
127.0.0.1:6379> set key1 value1
OK
127.0.0.1:6379> set key2 value2
OK
# check the inserted data
127.0.0.1:6379> get key1
"value1"
127.0.0.1:6379> get key2
"value2"
# exit from redis-cli
127.0.0.1:6379> exit
```

We have successfully deployed a Redis database and inserted some sample data into it. In the subsequent sections, we are going to backup these data using Stash.

## Prepare for Backup

In this section, we are going to prepare the necessary resources (i.e. database connection information, backend information, etc.) before backup.

### Ensure Redis Addon

When you install Stash Enterprise version, it will automatically install all the official database addons. Make sure that Redis addon was installed properly using the following command.

```bash
❯ kubectl get tasks.stash.appscode.com | grep redis
redis-backup-6.2.5            24m
redis-restore-6.2.5           24m
```

This addon should be able to take backup of the databases with matching major versions as discussed in [Addon Version Compatibility](/docs/addons/redis/README.md#addon-version-compatibility).

### Create AppBinding

Stash needs to know how to connect with the database. An `AppBinding` exactly provides this information. It holds the Service and Secret information of the database. You have to point to the respective `AppBinding` as a target of backup instead of the database itself.

Stash expect your database Secret to have `password` keys. If your database secret does not have the expected key, the `AppBinding` can also help here. You can specify a `secretTransforms` section with the mapping between the current keys and the desired keys.

Here, is the YAML of the `AppBinding` that we are going to create for the Redis database we have deployed earlier.

```yaml
apiVersion: appcatalog.appscode.com/v1alpha1
kind: AppBinding
metadata:
  name: sample-redis
  namespace: demo
spec:
  clientConfig:
    service:
      name: sample-redis-master
      path: /
      port: 6379
      scheme: http
  secret:
    name: sample-redis
  secretTransforms:
  - renameKey:
      from: redis-password
      to: password
  type: redis
  version: 6.2.5
```

Here,

- **.spec.clientConfig.service** specifies the Service information to use to connects with the database.
- **.spec.secret** specifies the name of the Secret that holds necessary credentials to access the database. If your Redis is not using authentication, then don't provide this field.
- **.spec.secretTransforms** specifies the transformations required to achieve the desired keys from the current Secret. You can apply the following transformations here:
  - **addKey**: If your database Secret does not have an equivalent key expected by Stash, you can add the key using `addKey` transformation.
  - **renameKey**: If your database Secret does not have a key expected by Stash but it has an equivalent key that is used for the same purpose, you can use `renameKey` transformation to specify the mapping between the keys. For example, our Redis Secret didn't have `password` key but it has an equivalent `redis-password` key that contains password for the database. Hence, we are telling Stash using `renameKey` transformation that the `redis-password` should be used as `password` key.
  - **addKeysFrom**: You can also merge keys from another Secret using `addKeysFrom` transformation. You have to specify the respective Secret name and namespace as below:
    ```yaml
    addKeysFrom:
      name: <secret name>
      namespace: <secret namespace>
    ```
- `spec.type` specifies the type of the database. This is particularly helpful in auto-backup where you want to use different path prefixes for different types of database.

Let's create the `AppBinding` we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/tree/{{< param "info.version" >}}/docs/addons/redis/helm/examples/appbinding.yaml
appbinding.appcatalog.appscode.com/sample-redis created
```

>The `secretTransforms` does not modify your original database Secret. Stash just uses those transformations to obtain the desired keys from the original Secret.

### Prepare Backend

We are going to store our backed up data into a GCS bucket. So, we need to create a Secret with GCS credentials and a `Repository` object with the bucket information. If you want to use a different backend, please read the respective backend configuration doc from [here](/docs/guides/backends/overview.md).

**Create Storage Secret:**

At first, let's create a secret called `gcs-secret` with access credentials to our desired GCS bucket,

```bash
$ echo -n 'changeit' > RESTIC_PASSWORD
$ echo -n '<your-project-id>' > GOOGLE_PROJECT_ID
$ cat downloaded-sa-json.key > GOOGLE_SERVICE_ACCOUNT_JSON_KEY
$ kubectl create secret generic -n demo gcs-secret \
    --from-file=./RESTIC_PASSWORD \
    --from-file=./GOOGLE_PROJECT_ID \
    --from-file=./GOOGLE_SERVICE_ACCOUNT_JSON_KEY
secret/gcs-secret created
```

**Create Repository:**

Now, crete a `Repository` object with the information of your desired bucket. Below is the YAML of `Repository` object we are going to create,

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
      prefix: /demo/redis/sample-redis
    storageSecretName: gcs-secret
```

Let's create the `Repository` we have shown above,

```bash
$ kubectl create -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/addons/redis/helm/examples/repository.yaml
repository.stash.appscode.com/gcs-repo created
```

Now, we are ready to backup our database into our desired backend.

### Backup

To schedule a backup, we have to create a `BackupConfiguration` object targeting the respective `AppBinding` of our desired database. Then Stash will create a CronJob to periodically backup the database.

#### Create BackupConfiguration

Below is the YAML for `BackupConfiguration` object we care going to use to backup the `sample-redis` database we have deployed earlier,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: sample-redis-backup
  namespace: demo
spec:
  schedule: "*/5 * * * *"
  task:
    name: redis-backup-6.2.5
  repository:
    name: gcs-repo
  target:
    ref:
      apiVersion: appcatalog.appscode.com/v1alpha1
      kind: AppBinding
      name: sample-redis
  retentionPolicy:
    name: keep-last-5
    keepLast: 5
    prune: true
```

Here,

- `.spec.schedule` specifies that we want to backup the database at 5 minutes intervals.
- `.spec.task.name` specifies the name of the Task object that specifies the necessary Functions and their execution order to backup a Redis database.
- `.spec.repository.name` specifies the Repository CR name we have created earlier with backend information.
- `.spec.target.ref` refers to the AppBinding object that holds the connection information of our targeted database.
- `.spec.retentionPolicy` specifies a policy indicating how we want to cleanup the old backups.

Let's create the `BackupConfiguration` object we have shown above,

```bash
$ kubectl create -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/addons/redis/helm/examples/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/sample-redis-backup created
```


#### Verify Backup Setup Successful

If everything goes well, the Phase of the `BackupConfiguration` should be `Ready`. The `Ready` Phase indicates that the backup setup is successful. Let's verify the `Phase` of the BackupConfiguration,

```bash
$ kubectl get backupconfiguration -n demo
NAME                  TASK                       SCHEDULE      PAUSED   PHASE      AGE
sample-redis-backup   redis-backup-6.2.5         */5 * * * *            Ready      11s
```

> If the BackupConfiguration is not in `Ready` state, you need to describe that CRD for finding out the specific reason of the backup setup being unsuccessful.  Describe the BackupConfiguration by following command,
```bash
$ kubectl describe backupconfiguration -n demo sample-redis-backup
```


#### Verify CronJob

If everything goes well, Stash will create a CronJob with the schedule specified in `spec.schedule` field of `BackupConfiguration` object.

Verify that the CronJob has been created using the following command,

```bash
❯ kubectl get cronjob -n demo
NAME                               SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
stash-backup-sample-redis-backup   */5 * * * *   False     0        <none>          14s
```

#### Wait for BackupSession

The `sample-redis-backup` CronJob will trigger a backup on each scheduled slot by creating a `BackupSession` object.

Now, wait for a schedule to appear. Run the following command to watch for a `BackupSession` object,

```bash
❯ kubectl get backupsession -n demo -w
NAME                             INVOKER-TYPE          INVOKER-NAME          PHASE       DURATION          AGE
sample-redis-backup-1627490702   BackupConfiguration   sample-redis-backup                                 0s
sample-redis-backup-1627490702   BackupConfiguration   sample-redis-backup   Running                       0s
sample-redis-backup-1627490702   BackupConfiguration   sample-redis-backup   Succeeded   1m18.098555424s   78s
```

Here, the phase `Succeeded` means that the backup process has been completed successfully.

#### Verify Backup

Now, we are going to verify whether the backed up data is present in the backend or not. Once a backup is completed, Stash will update the respective `Repository` object to reflect the backup completion. Check that the repository `gcs-repo` has been updated by the following command,

```bash
❯ kubectl get repository -n demo
NAME       INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
gcs-repo   true        93 B   1                2m1s                     24m
```

Now, if we navigate to the GCS bucket, we will see the backed up data has been stored in `demo/redis/sample-redis` directory as specified by `.spec.backend.gcs.prefix` field of the `Repository` object.

<figure align="center">
  <img alt="Backup data in GCS Bucket" src="/docs/addons/redis/helm/images/sample-redis-backup.png">
  <figcaption align="center">Fig: Backup data in GCS Bucket</figcaption>
</figure>

> Note: Stash keeps all the backed up data encrypted. So, data in the backend will not make any sense until they are decrypted.

## Restore Redis

If you have followed the previous sections properly, you should have a successful logical backup of your Redis database. Now, we are going to show how you can restore the database from the backed up data.

### Restore Into the Same Database

You can restore your data into the same database you have backed up from or into a different database in the same cluster or a different cluster. In this section, we are going to show you how to restore in the same database which may be necessary when you have accidentally deleted any data from the running database.

#### Temporarily Pause Backup

At first, let's stop taking any further backup of the database so that no backup runs after we delete the sample data. We are going to pause the `BackupConfiguration` object. Stash will stop taking any further backup when the `BackupConfiguration` is paused.

Let's pause the `sample-redis-backup` BackupConfiguration,

```bash
$ kubectl patch backupconfiguration -n demo sample-redis-backup --type="merge" --patch='{"spec": {"paused": true}}'
backupconfiguration.stash.appscode.com/sample-redis-backup patched
```

Verify that the `BackupConfiguration` has been paused,

```bash
❯ kubectl get backupconfiguration -n demo sample-redis-backup
NAME                  TASK                 SCHEDULE      PAUSED   PHASE   AGE
sample-redis-backup   redis-backup-6.2.5   */5 * * * *   true     Ready   4h47m
```

Notice the `PAUSED` column. Value `true` for this field means that the `BackupConfiguration` has been paused.

Stash will also suspend the respective CronJob.

```bash
❯ kubectl get cronjob -n demo
NAME                               SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
stash-backup-sample-redis-backup   */5 * * * *   True      0        113s            4h48m
```

#### Simulate Disaster

Now, let's simulate an accidental deletion scenario. Here, we are going to exec into the database pod and delete the sample data we have inserted earlier.

```bash
❯ kubectl exec -it -n demo sample-redis-master-0 -- redis-cli -a $PASSWORD
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
# delete the sample data
127.0.0.1:6379> del key1 key2
(integer) 2
# verify that the sample data has been deleted
127.0.0.1:6379> get key1
(nil)
127.0.0.1:6379> get key2
(nil)
127.0.0.1:6379> exit
```

#### Create RestoreSession

To restore the database, you have to create a `RestoreSession` object pointing to the `AppBinding` of the targeted database.

Here, is the YAML of the `RestoreSession` object that we are going to use for restoring our `sample-redis` database.

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: sample-redis-restore
  namespace: demo
spec:
  task:
    name: redis-restore-6.2.5
  repository:
    name: gcs-repo
  target:
    ref:
      apiVersion: appcatalog.appscode.com/v1alpha1
      kind: AppBinding
      name: sample-redis
  rules:
  - snapshots: [latest]
```

Here,

- `.spec.task.name` specifies the name of the Task object that specifies the necessary Functions and their execution order to restore a Redis database.
- `.spec.repository.name` specifies the Repository object that holds the backend information where our backed up data has been stored.
- `.spec.target.ref` refers to the respective AppBinding of the `sample-redis` database.
- `.spec.rules` specifies that we are restoring data from the latest backup snapshot of the database.

Let's create the `RestoreSession` object object we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/addons/redis/helm/examples/restoresession.yaml
restoresession.stash.appscode.com/sample-redis-restore created
```

Once, you have created the `RestoreSession` object, Stash will create a restore Job. Run the following command to watch the phase of the `RestoreSession` object,

```bash
❯ kubectl get restoresession -n demo -w
NAME                   REPOSITORY   PHASE     DURATION          AGE
sample-redis-restore   gcs-repo     Running                     6s
sample-redis-restore   gcs-repo     Running                     16s
sample-redis-restore   gcs-repo     Succeeded                   16s
sample-redis-restore   gcs-repo     Succeeded   16.324570911s   16s
```

The `Succeeded` phase means that the restore process has been completed successfully.

#### Verify Restored Data

Now, let's exec into the database pod and verify whether data actual data has been restored or not,

```bash
❯ kubectl exec -it -n demo sample-redis-master-0 -- redis-cli -a $PASSWORD
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
127.0.0.1:6379> get key1
"value1"
127.0.0.1:6379> get key2
"value2"
127.0.0.1:6379> exit
```

Hence, we can see from the above output that the deleted data has been restored successfully from the backup.

#### Resume Backup

Since our data has been restored successfully we can now resume our usual backup process. Resume the `BackupConfiguration` using following command,

```bash
❯ kubectl patch backupconfiguration -n demo sample-redis-backup --type="merge" --patch='{"spec": {"paused": false}}'
backupconfiguration.stash.appscode.com/sample-redis-backup patched
```

Verify that the `BackupConfiguration` has been resumed,
```bash
❯ kubectl get backupconfiguration -n demo sample-redis-backup
NAME                  TASK                 SCHEDULE      PAUSED   PHASE   AGE
sample-redis-backup   redis-backup-6.2.5   */5 * * * *   false    Ready   4h54m
```

Here,  `false` in the `PAUSED` column means the backup has been resume successfully. The CronJob also should be resumed now.

```bash
❯ kubectl get cronjob -n demo
NAME                               SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
stash-backup-sample-redis-backup   */5 * * * *   False     0        3m24s           4h54m
```

Here, `False` in the `SUSPEND` column means the CronJob is no longer suspended and will trigger in the next schedule.

### Restore Into Different Database of the Same Namespace

If you want to restore the backed up data into a different database of the same namespace, you have to create another `AppBinding` pointing to the desired database. Then, you have to create the `RestoreSession` pointing to the new `AppBinding`.

### Restore Into Different Namespace

If you want to restore into a different namespace of the same cluster, you have to create the Repository, backend Secret, AppBinding, in the desired namespace. You can use [Stash kubectl plugin](https://stash.run/docs/{{< param "info.version" >}}/guides/cli/cli/) to easily copy the resources into a new namespace. Then, you have to create the `RestoreSession` object in the desired namespace pointing to the Repository, AppBinding of that namespace.

### Restore Into Different Cluster

If you want to restore into a different cluster, you have to install Stash in the desired cluster. Then, you have to install Stash Redis addon in that cluster too. Then, you have to create the Repository, backend Secret, AppBinding, in the desired cluster. Finally, you have to create the `RestoreSession` object in the desired cluster pointing to the Repository, AppBinding of that cluster.

## Cleanup

To cleanup the Kubernetes resources created by this tutorial, run:

```bash
kubectl delete -n demo backupconfiguration sample-redis-backup
kubectl delete -n demo restoresession sample-redis-restore
kubectl delete -n demo repository gcs-repo
# delete the database chart
helm delete sample-redis -n demo
```
