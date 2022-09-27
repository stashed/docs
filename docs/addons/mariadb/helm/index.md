---
title: Logical Backup & Restore MariaDB | Stash
description: Take logical backup of MariaDB database using Stash
menu:
  docs_{{ .version }}:
    identifier: stash-mariadb-helm
    name: Helm managed MariaDB
    parent: stash-mariadb
    weight: 20
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: stash-addons
---

# Take a logical backup of the MariaDB database using Stash

Stash `v0.11.8+` supports backup and restoration of MariaDB databases. This guide will show you how you can take a logical backup of your MariaDB databases and restore them using Stash.

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster.
- Install Stash Enterprise in your cluster following the steps [here](/docs/setup/install/enterprise/index.md).
- If you are not familiar with how Stash backup and restore MariaDB databases, please check the following guide [here](/docs/addons/mariadb/overview/index.md).

You have to be familiar with following custom resources:

- [AppBinding](/docs/concepts/crds/appbinding/index.md)
- [Function](/docs/concepts/crds/function/index.md)
- [Task](/docs/concepts/crds/task/index.md)
- [BackupConfiguration](/docs/concepts/crds/backupconfiguration/index.md)
- [BackupSession](/docs/concepts/crds/backupsession/index.md)
- [RestoreSession](/docs/concepts/crds/restoresession/index.md)

To keep things isolated, we are going to use a separate namespace called `demo` throughout this tutorial. Create `demo` namespace if you haven't created it yet.

```bash
$ kubectl create ns demo
namespace/demo created
```

> Note: YAML files used in this tutorial are stored [here](https://github.com/stashed/docs/tree/{{< param "info.version" >}}/docs/addons/mariadb/helm/examples).

## Prepare MariaDB

In this section, we are going to deploy a MariaDB database. Then, we are going to insert some sample data into it.

### Deploy MariaDB

At first, let's deploy a MariaDB database. Here, we are going to use [bitnami/mariadb](https://artifacthub.io/packages/helm/bitnami/mariadb)  chart from [ArtifactHub](https://artifacthub.io/).

Let's deploy a MariaDB database named `sample-mariadb` using Helm as below,

```bash
# Add bitnami chart registry
$ helm repo add bitnami https://charts.bitnami.com/bitnami
# Update helm registries
$ helm repo update
# Install bitnami/mariadb chart into demo namespace
$ helm install sample-mariadb bitnami/mariadb -n demo
```

This chart will create the necessary StatefulSet, Secret, Service etc. for the database. You can easily view all the resources created by chart using [ketall](https://github.com/corneliusweig/ketall) `kubectl` plugin as below,

```bash
$ kubectl get-all -n demo -l app.kubernetes.io/instance=sample-mariadb
NAME                                              NAMESPACE  AGE
configmap/sample-mariadb                          demo       5m59s  
endpoints/sample-mariadb                          demo       5m59s  
persistentvolumeclaim/data-sample-mariadb-0       demo       5m59s  
pod/sample-mariadb-0                              demo       5m59s  
secret/sample-mariadb                             demo       5m59s  
serviceaccount/sample-mariadb                     demo       5m59s  
service/sample-mariadb                            demo       5m59s  
controllerrevision.apps/sample-mariadb-bb8d8865b  demo       5m59s  
statefulset.apps/sample-mariadb                   demo       5m59s
```

Now, wait for the database pod `sample-mariadb-0` to go into `Running` state,

```bash
$ kubectl get pod -n demo sample-mariadb-0
NAME               READY   STATUS    RESTARTS   AGE
sample-mariadb-0   1/1     Running   0          11m
```

Once the database pod is in `Running` state, verify that the database is ready to accept the connections.

```bash
$ kubectl logs -n demo sample-mariadb-0
mariadb 10:44:37.29 
mariadb 10:44:37.29 Welcome to the Bitnami mariadb container
...
2020-12-03 10:44:46 0 [Note] /opt/bitnami/mariadb/sbin/mysqld: ready for connections.
Version: '10.5.8-MariaDB'  socket: '/opt/bitnami/mariadb/tmp/mysql.sock'  port: 3306  Source distribution
```

From the above log, we can see the database is ready to accept connections.

### Insert Sample Data

Now, we are going to exec into the database pod and create some sample data. The helm chart has created a secret with access credentials. Let's find out the credentials from the Secret,

```yaml
$ kubectl get secret -n demo sample-mariadb -o yaml
apiVersion: v1
data:
  mariadb-password: ZlFSdzA1ZXRvbg==
  mariadb-root-password: Y1ZrUXA0TXdENQ==
kind: Secret
metadata:
  annotations:
    meta.helm.sh/release-name: sample-mariadb
    meta.helm.sh/release-namespace: demo
  creationTimestamp: "2020-12-03T10:43:43Z"
  labels:
    app.kubernetes.io/instance: sample-mariadb
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: mariadb
    helm.sh/chart: mariadb-9.0.1
  ....
  name: sample-mariadb
  namespace: demo
type: Opaque
```

Here, we are going to use the root user credential `mariadb-root-password` to insert the sample data.

At first, let's export the username and password as environment variables to make further commands re-usable.

```bash
export USER_NAME=root
export PASSWORD=$(kubectl get secrets -n demo sample-mariadb -o jsonpath='{.data.\mariadb-root-password}' | base64 -d)
```

Now, let's exec into the database pod and insert some sample data,

```bash
$ kubectl exec -it -n demo sample-mariadb-0 -- mariadb --user=$USER_NAME --password=$PASSWORD
...
# Let's create a database named "company"
MariaDB [(none)]> create database company;
Query OK, 1 row affected (0.001 sec)

# Verify that the database has been created successfully
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| company            |
| information_schema |
| my_database        |
| mysql              |
| performance_schema |
| test               |
+--------------------+
6 rows in set (0.001 sec)

# Now, let create a table called "employee" in the "company" table
MariaDB [(none)]> create table company.employees ( name varchar(50), salary int);
Query OK, 0 rows affected (0.018 sec)

# Verify that the table has been created successfully
MariaDB [(none)]> show tables in company;
+-------------------+
| Tables_in_company |
+-------------------+
| employees         |
+-------------------+
1 row in set (0.001 sec)

# Now, let's insert a sample row in the table
MariaDB [(none)]> insert into company.employees values ('John Doe', 5000);
Query OK, 1 row affected (0.003 sec)

# Insert another sample row
MariaDB [(none)]> insert into company.employees values ('James William', 7000);
Query OK, 1 row affected (0.002 sec)

# Verify that the rows have been inserted into the table successfully
MariaDB [(none)]> select * from company.employees;
+---------------+--------+
| name          | salary |
+---------------+--------+
| John Doe      |   5000 |
| James William |   7000 |
+---------------+--------+
2 rows in set (0.001 sec)

MariaDB [(none)]> exit
Bye
```

We have successfully deployed a MariaDB database and inserted some sample data into it. In the subsequent sections, we are going to backup these data using Stash.

## Prepare for Backup

In this section, we are going to prepare the necessary resources (i.e. database connection information, backend information, etc.) before backup.

### Ensure MariaDB Addon

When you install Stash Enterprise version, it will automatically install all the official database addons. Make sure that MariaDB addon was installed properly using the following command.

```bash
$ kubectl get tasks.stash.appscode.com | grep mariadb
mariadb-backup-10.5.8   35s
mariadb-backup-10.5.8   35s
```

This addon should be able to take backup of the databases with matching major versions as discussed in [Addon Version Compatibility](/docs/addons/mariadb/README.md#addon-version-compatibility).

### Create AppBinding

Stash needs to know how to connect with the database. An `AppBinding` exactly provides this information. It holds the Service and Secret information of the database. You have to point to the respective `AppBinding` as a target of backup instead of the database itself.

Stash expect your database Secret to have `username` and `password` keys. If your database secret does not have them, the `AppBinding` can also help here. You can specify a `secretTransforms` section with the mapping between the current keys and the desired keys.

Here, is the YAML of the `AppBinding` that we are going to create for the MariaDB database we have deployed earlier.

```yaml
apiVersion: appcatalog.appscode.com/v1alpha1
kind: AppBinding
metadata:
  name: sample-mariadb
  namespace: demo
spec:
  clientConfig:
    service:
      name: sample-mariadb
      path: /
      port: 3306
      scheme: mysql
  secret:
    name: sample-mariadb
  secretTransforms:
  - addKey:
      key: username
      stringValue: root
  - renameKey:
      from: mariadb-root-password
      to: password
  type: mariadb
  version: 10.5.8
```

Here,

- **.spec.clientConfig.service** specifies the Service information to use to connects with the database.
- **.spec.secret** specifies the name of the Secret that holds necessary credentials to access the database.
- **.spec.secretTransforms** specifies the transformations required to achieve the desired keys from the current Secret. You can apply the following transformations here:
  - **addKey**: If your database Secret does not have an equivalent key expected by Stash, you can add the key using `addKey` transformation. Here, our deployed MariaDB Secret didn't have any key equivalent to `username`. Hence, we are adding the key using `addKey` transformation.
  - **renameKey**: If your database Secret does not have a key expected by Stash but it has an equivalent key that is used for the same purpose, you can use `renameKey` transformation to specify the mapping between the keys. For example, our MariaDB Secret didn't have `password` key but it has an equivalent `mariadb-root-password` key that contains password for the root user. Hence, we are telling Stash using `renameKey` transformation that the `mariadb-root-password` should be used as `password` key.
  - **addKeysFrom**: You can also merge keys from another Secret using `addKeysFrom` transformation. You have to specify the respective Secret name and namespace as below:
    ```yaml
    addKeysFrom:
      name: <secret name>
      namespace: <secret namespace>
    ```
- `spec.type` specifies the type of the database. This is particularly helpful in auto-backup where you want to use different path prefixes for different types of database.

Let's create the `AppBinding` we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/tree/{{< param "info.version" >}}/docs/addons/mariadb/helm/examples/appbinding.yaml
appbinding.appcatalog.appscode.com/sample-mariadb created
```

>The `secretTransforms` does not modify your original database Secret. Stash just uses those transformations to obtain the desired keys from the original Secret.

### Prepare Backend

We are going to store our backed up data into a GCS bucket. So, we need to create a Secret with GCS credentials and a `Repository` object with the bucket information. If you want to use a different backend, please read the respective backend configuration doc from [here](/docs/guides/backends/overview/index.md).

**Create Storage Secret:**

At first, let's create a secret called `gcs-secret` with access credentials to our desired GCS bucket,

```bash
$ echo -n 'changeit' > RESTIC_PASSWORD
$ echo -n '<your-project-id>' > GOOGLE_PROJECT_ID
$ cat downloaded-sa-key.json > GOOGLE_SERVICE_ACCOUNT_JSON_KEY
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
      prefix: /demo/mariadb/sample-mariadb
    storageSecretName: gcs-secret
```

Let's create the `Repository` we have shown above,

```bash
$ kubectl create -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/addons/mariadb/helm/examples/repository.yaml
repository.stash.appscode.com/gcs-repo created
```

Now, we are ready to backup our database into our desired backend.

### Backup

To schedule a backup, we have to create a `BackupConfiguration` object targeting the respective `AppBinding` of our desired database. Then Stash will create a CronJob to periodically backup the database.

#### Create BackupConfiguration

Below is the YAML for `BackupConfiguration` object we care going to use to backup the `sample-mariadb` database we have deployed earlier,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: sample-mariadb-backup
  namespace: demo
spec:
  schedule: "*/5 * * * *"
  task:
    name: mariadb-backup-10.5.8
  repository:
    name: gcs-repo
  target:
    ref:
      apiVersion: appcatalog.appscode.com/v1alpha1
      kind: AppBinding
      name: sample-mariadb
  retentionPolicy:
    name: keep-last-5
    keepLast: 5
    prune: true
```

Here,

- `.spec.schedule` specifies that we want to backup the database at 5 minutes intervals.
- `.spec.task.name` specifies the name of the Task object that specifies the necessary Functions and their execution order to backup a MariaDB database.
- `.spec.target.ref` refers to the AppBinding object that holds the connection information of our targeted database.

Let's create the `BackupConfiguration` object we have shown above,

```bash
$ kubectl create -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/addons/mariadb/helm/examples/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/sample-mariadb-backup created
```

### Verify Backup Setup Successful

If everything goes well, the phase of the `BackupConfiguration` should be `Ready`. The `Ready` phase indicates that the backup setup is successful. Let's verify the `Phase` of the BackupConfiguration,

```bash
$ kubectl get backupconfiguration -n demo
NAME                    TASK                    SCHEDULE      PAUSED   PHASE      AGE
sample-mariadb-backup   mariadb-backup-10.5.8   */5 * * * *            Ready      11s
```

#### Verify CronJob

Stash will create a CronJob with the schedule specified in `spec.schedule` field of `BackupConfiguration` object.

Verify that the CronJob has been created using the following command,

```bash
$ kubectl get cronjob -n demo
NAME                                 SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
stash-backup-sample-mariadb-backup   */5 * * * *   False     0        15s             17s
```

#### Wait for BackupSession

The `sample-mariadb-backup` CronJob will trigger a backup on each scheduled slot by creating a `BackupSession` object.

Now, wait for a schedule to appear. Run the following command to watch for a `BackupSession` object,

```bash
$ kubectl get backupsession -n demo -w
NAME                               INVOKER-TYPE          INVOKER-NAME            PHASE     AGE
sample-mariadb-backup-1606994706   BackupConfiguration   sample-mariadb-backup   Running   24s
sample-mariadb-backup-1606994706   BackupConfiguration   sample-mariadb-backup   Running   75s
sample-mariadb-backup-1606994706   BackupConfiguration   sample-mariadb-backup   Succeeded   103s
```

Here, the phase `Succeeded` means that the backup process has been completed successfully.

#### Verify Backup

Now, we are going to verify whether the backed up data is present in the backend or not. Once a backup is completed, Stash will update the respective `Repository` object to reflect the backup completion. Check that the repository `gcs-repo` has been updated by the following command,

```bash
$ kubectl get repository -n demo gcs-repo
NAME       INTEGRITY   SIZE        SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
gcs-repo   true        1.327 MiB   1                60s                      8m
```

Now, if we navigate to the GCS bucket, we will see the backed up data has been stored in `demo/mariadb/sample-mariadb` directory as specified by `.spec.backend.gcs.prefix` field of the `Repository` object.

<figure align="center">
  <img alt="Backup data in GCS Bucket" src="/docs/addons/mariadb/helm/images/sample-mariadb-backup.png">
  <figcaption align="center">Fig: Backup data in GCS Bucket</figcaption>
</figure>

> Note: Stash keeps all the backed up data encrypted. So, data in the backend will not make any sense until they are decrypted.

## Restore MariaDB

If you have followed the previous sections properly, you should have a successful logical backup of your MariaDB database. Now, we are going to show how you can restore the database from the backed up data.

### Restore Into the Same Database

You can restore your data into the same database you have backed up from or into a different database in the same cluster or a different cluster. In this section, we are going to show you how to restore in the same database which may be necessary when you have accidentally deleted any data from the running database.

#### Temporarily Pause Backup

At first, let's stop taking any further backup of the database so that no backup runs after we delete the sample data. We are going to pause the `BackupConfiguration` object. Stash will stop taking any further backup when the `BackupConfiguration` is paused.

Let's pause the `sample-mariadb-backup` BackupConfiguration,

```bash
$ kubectl patch backupconfiguration -n demo sample-mariadb-backup --type="merge" --patch='{"spec": {"paused": true}}'
backupconfiguration.stash.appscode.com/sample-mariadb-backup patched
```

Verify that the `BackupConfiguration` has been paused,

```bash
$ kubectl get backupconfiguration -n demo sample-mariadb-backup
NAME                   TASK                    SCHEDULE      PAUSED   PHASE   AGE
sample-mariadb-backup  mariadb-backup-10.5.8   */5 * * * *   true     Ready   26m
```

Notice the `PAUSED` column. Value `true` for this field means that the `BackupConfiguration` has been paused.

Stash will also suspend the respective CronJob.

```bash
$ kubectl get cronjob -n demo
NAME                                 SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
stash-backup-sample-mariadb-backup   */5 * * * *   True      0        2m59s           20m
```

#### Simulate Disaster

Now, let's simulate an accidental deletion scenario. Here, we are going to exec into the database pod and delete the `company` database we had created earlier.

```bash
$ kubectl exec -it -n demo sample-mariadb-0 -- mariadb --user=$USER_NAME --password=$PASSWORD

# View current databases
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| company            |
| information_schema |
| my_database        |
| mysql              |
| performance_schema |
| test               |
+--------------------+
6 rows in set (0.001 sec)

# Let's delete the "company" database
MariaDB [(none)]> drop database company;
Query OK, 1 row affected (0.268 sec)

# Verify that the "company" database has been deleted
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| my_database        |
| mysql              |
| performance_schema |
| test               |
+--------------------+
5 rows in set (0.001 sec)

MariaDB [(none)]> exit
Bye
```

#### Create RestoreSession

To restore the database, you have to create a `RestoreSession` object pointing to the `AppBinding` of the targeted database.

Here, is the YAML of the `RestoreSession` object that we are going to use for restoring our `sample-mariadb` database.

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: sample-mariadb-restore
  namespace: demo
spec:
  task:
    name: mariadb-backup-10.5.8
  repository:
    name: gcs-repo
  target:
    ref:
      apiVersion: appcatalog.appscode.com/v1alpha1
      kind: AppBinding
      name: sample-mariadb
  rules:
  - snapshots: [latest]
```

Here,

- `.spec.task.name` specifies the name of the Task object that specifies the necessary Functions and their execution order to restore a MariaDB database.
- `.spec.repository.name` specifies the Repository object that holds the backend information where our backed up data has been stored.
- `.spec.target.ref` refers to the respective AppBinding of the `sample-mariadb` database.
- `.spec.rules` specifies that we are restoring data from the latest backup snapshot of the database.

Let's create the `RestoreSession` object object we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/addons/mariadb/helm/examples/restoresession.yaml
restoresession.stash.appscode.com/sample-mariadb-restore created
```

Once, you have created the `RestoreSession` object, Stash will create a restore Job. Run the following command to watch the phase of the `RestoreSession` object,

```bash
$ kubectl get restoresession -n demo -w
NAME                     REPOSITORY   PHASE     AGE
sample-mariadb-restore   gcs-repo     Running   15s
sample-mariadb-restore   gcs-repo     Succeeded   18s
```

The `Succeeded` phase means that the restore process has been completed successfully.

#### Verify Restored Data

Now, let's exec into the database pod and verify whether data actual data was restored or not,

```bash
$ kubectl exec -it -n demo sample-mariadb-0 -- mariadb --user=$USER_NAME --password=$PASSWORD

# Verify that the "company" database has been restored
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| company            |
| information_schema |
| my_database        |
| mysql              |
| performance_schema |
| test               |
+--------------------+
6 rows in set (0.000 sec)

# Verify that the tables of the "company" database have been restored
MariaDB [(none)]> show tables from company;
+-------------------+
| Tables_in_company |
+-------------------+
| employees         |
+-------------------+
1 row in set (0.000 sec)

# Verify that the sample data of the "employees" table has been restored
MariaDB [(none)]> select * from company.employees;
+---------------+--------+
| name          | salary |
+---------------+--------+
| John Doe      |   5000 |
| James William |   7000 |
+---------------+--------+
2 rows in set (0.000 sec)

MariaDB [(none)]> exit
Bye
```

Hence, we can see from the above output that the deleted data has been restored successfully from the backup.

#### Resume Backup

Since our data has been restored successfully we can now resume our usual backup process. Resume the `BackupConfiguration` using following command,

```bash
$ kubectl patch backupconfiguration -n demo sample-mariadb-backup --type="merge" --patch='{"spec": {"paused": false}}'
backupconfiguration.stash.appscode.com/sample-mariadb-backup patched
```

Verify that the `BackupConfiguration` has been resumed,
```bash
$ kubectl get backupconfiguration -n demo sample-mariadb-backup
NAME                    TASK                         SCHEDULE      PAUSED   PHASE   AGE
sample-mariadb-backup   mariadb-backup-10.5.8        */5 * * * *   false    Ready   29m
```

Here,  `false` in the `PAUSED` column means the backup has been resume successfully. The CronJob also should be resumed now.

```bash
$ kubectl get cronjob -n demo
NAME                                 SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
stash-backup-sample-mariadb-backup   */5 * * * *   False     0        2m59s           29m
```

Here, `False` in the `SUSPEND` column means the CronJob is no longer suspended and will trigger in the next schedule.

### Restore Into Different Database of the Same Namespace

If you want to restore the backed up data into a different database of the same namespace, you have to create another `AppBinding` pointing to the desired database. Then, you have to create the `RestoreSession` pointing to the new `AppBinding`.

### Restore Into Different Namespace

If you want to restore into a different namespace of the same cluster, you have to create the Repository, backend Secret, AppBinding, in the desired namespace. You can use [Stash kubectl plugin](https://stash.run/docs/{{< param "info.version" >}}/guides/cli/cli/) to easily copy the resources into a new namespace. Then, you have to create the `RestoreSession` object in the desired namespace pointing to the Repository, AppBinding of that namespace.

### Restore Into Different Cluster

If you want to restore into a different cluster, you have to install Stash in the desired cluster. Then, you have to install Stash MariaDB addon in that cluster too. Then, you have to create the Repository, backend Secret, AppBinding, in the desired cluster. Finally, you have to create the `RestoreSession` object in the desired cluster pointing to the Repository, AppBinding of that cluster.

## Cleanup

To cleanup the Kubernetes resources created by this tutorial, run:

```bash
kubectl delete -n demo backupconfiguration sample-mariadb-backup
kubectl delete -n demo restoresession sample-mariadb-restore
kubectl delete -n demo repository gcs-repo
# delete the database chart
helm delete sample-mariadb -n demo
```
