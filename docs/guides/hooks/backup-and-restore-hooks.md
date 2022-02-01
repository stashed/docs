---
title: Backup & Restore Hooks | Stash
menu:
  docs_{{ .version }}:
    identifier: backup-and-restore-hooks
    name: Backup & Restore Hooks
    parent: hooks
    weight: 20
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: guides
---

# Backup & Restore Hooks

Stash hooks let you perform some actions before and after the backup or restore process. This is particularly helpful when you want to prepare your application before backup or restore.

Here, we are going to demonstrate how you can perform different actions before and after backup and restore a MySQL database. Some of the examples might not reflect the real-world use cases but it serves the sole purpose of demonstrating what is possible.

> Note that, this is an advanced concept. If you haven't tried the normal backup restore processes yet, we will recommend to try them first.

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [kind](https://kind.sigs.k8s.io/docs/user/quick-start/).
- Install Stash Enterprise in your cluster following the steps [here](/docs/setup/install/enterprise.md).
- Install [KubeDB](https://kubedb.com) in your cluster following the steps [here](https://kubedb.com/docs/latest/setup/). This step is optional. You can deploy your database using any method you want. We are using KubeDB because KubeDB simplifies many of the difficult or tedious management tasks of running production-grade databases on private and public clouds.
- If you are not familiar with how Stash backup and restore MySQL databases, please check the following guide [here](/docs/addons/mysql/overview/index.md).
- Also, if you haven't read about how hooks work in Stash, please check it from [here](/docs/guides/hooks/overview.md).

You should be familiar with the following `Stash` concepts:

- [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md)
- [BackupSession](/docs/concepts/crds/backupsession.md)
- [Repository](/docs/concepts/crds/repository.md)
- [Function](/docs/concepts/crds/function.md)
- [Task](/docs/concepts/crds/task.md)
- [AppBinding](/docs/concepts/crds/appbinding.md)

To keep everything isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```bash
$ kubectl create ns demo
namespace/demo created
```

## Prepare Database

At first, let's deploy a MySQL database. Here, we are going to deploy MySQL `8.0.14` using KubeDB. We are going to insert some sample data into the database so that we can verify that the backup and restore process is working properly.

**Deploy Database:**

Below is the `MySQL` CR(Custom Resource) that we are going to create,

```yaml
apiVersion: kubedb.com/v1alpha2
kind: MySQL
metadata:
  name: sample-mysql
  namespace: demo
spec:
  version: 8.0.27
  replicas: 1
  storageType: Durable
  storage:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 1Gi
  terminationPolicy: WipeOut
```

Let's create the above `MySQL` CR,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/hooks/sample-mysql.yaml
mysql.kubedb.com/sample-mysql created
```

KubeDB will deploy a MySQL database according to the above specification. It will also create the necessary Secrets and Services to access the database.

Wait for the database to go into `Running` state,

```bash
$ kubectl get mysql -n demo -w
NAME           VERSION   STATUS    AGE
sample-mysql   8.0.14    Creating  5s
sample-mysql   8.0.14    Running   2m7s
```

**Verify Database Secret:**

Verify that KubeDB has created a Secret for the database.

```bash
$ kubectl get secret -n demo -l=app.kubernetes.io/instance=sample-mysql
NAME                TYPE     DATA   AGE
sample-mysql-auth   Opaque   2      5m7s
```

**Verify AppBinding:**

KubeDB creates an `AppBinding`  CR that holds the necessary information to connect with the database. Verify that the `AppBinding` has been created for the above database:

```bash
$ kubectl get appbindings -n demo -l=app.kubernetes.io/instance=sample-mysql
NAME           TYPE               VERSION   AGE
sample-mysql   kubedb.com/mysql   8.0.14    66s
```

If you check the YAML of the `AppBinding`, you will see the connection information and respective Secret reference to access the database is presents in `spec` section.

```bash
$ kubectl get appbindings sample-mysql -n demo -o yaml
```

```yaml
apiVersion: appcatalog.appscode.com/v1alpha1
kind: AppBinding
metadata:
  creationTimestamp: "2020-01-16T10:28:00Z"
  generation: 1
  labels:
    app.kubernetes.io/component: database
    app.kubernetes.io/instance: sample-mysql
    app.kubernetes.io/managed-by: kubedb.com
    app.kubernetes.io/name: mysql
    app.kubernetes.io/version: 8.0.14
    kubedb.com/kind: MySQL
    app.kubernetes.io/instance: sample-mysql
  name: sample-mysql
  namespace: demo
spec:
  clientConfig:
    service:
      name: sample-mysql
      path: /
      port: 3306
      scheme: mysql
    url: tcp(sample-mysql:3306)/
  secret:
    name: sample-mysql-auth
  type: kubedb.com/mysql
  version: 8.0.14
```

**Insert Sample Data:**

Now, let's insert some sample data into the above database. Here, we are going to `exec` into the database pod and create a database named `companyRecord`. Then, we are going to create a table named `employee` which will store employee's id, name and salary information. Then, we are going to insert a sample row in the table.

At first, let's export the database credentials as environment variables in our current shell so that we can use those variables to access the database instead of typing username and password every time.

```bash
# export username from the database secret
$ export MYSQL_USER=$(kubectl get secret -n demo  sample-mysql-auth -o jsonpath='{.data.username}'| base64 -d)

# verify that the username has been exported properly
$ echo $MYSQL_USER
root

# export the password from the database secret
$ export MYSQL_PASSWORD=$(kubectl get secret -n demo  sample-mysql-auth -o jsonpath='{.data.password}'| base64 -d)

# verify that the password has been exported properly
$ echo $MYSQL_PASSWORD
CWg2hru8b0Yu7dzS
```

Now, let's identify the database pod,

```bash
$ kubectl get pods -n demo --selector="app.kubernetes.io/instance=sample-mysql"
NAME             READY   STATUS    RESTARTS   AGE
sample-mysql-0   1/1     Running   0          6m50s
```

Let's `exec` into the database pod and insert sample data,

```bash
$ kubectl exec -it -n demo sample-mysql-0 -- mysql --user=$MYSQL_USER --password=$MYSQL_PASSWORD

mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 131
Server version: 8.0.14 MySQL Community Server - GPL

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

# create database named "companyRecord"
mysql> CREATE DATABASE companyRecord;
Query OK, 1 row affected (0.01 sec)

# verify that the database has been created
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| companyRecord      |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

# create a table called "employee" in "companyRecord" database
mysql> CREATE TABLE companyRecord.employee (id INT, name VARCHAR(50), salary INT, PRIMARY KEY(id));
Query OK, 0 rows affected (0.05 sec)

# insert a demo data into the table
mysql> INSERT INTO companyRecord.employee (id, name, salary) VALUES (1, "John Doe", 5000);
Query OK, 1 row affected (0.01 sec)

# verify that the data has been inserted
mysql> SELECT * FROM companyRecord.employee;
+----+----------+--------+
| id | name     | salary |
+----+----------+--------+
|  1 | John Doe |   5000 |
+----+----------+--------+
1 row in set (0.00 sec)

mysql> exit
Bye
```

### Prepare Backend

We are going to store our backed up data into a GCS bucket. At first, we need to create a secret with GCS credentials then we need to create a `Repository` CR. If you want to use a different backend, please read the respective backend configuration doc from [here](/docs/guides/backends/overview.md).

**Create Storage Secret:**

Let's create a secret called `gcs-secret` with access credentials to our desired GCS bucket,

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

Now, create a `Repository` using this secret. Below is the YAML of Repository CR we are going to create,

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
      prefix: /demo/mysql/hook-example
    storageSecretName: gcs-secret
```

Let's create the `Repository` we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/hooks/repository.yaml
repository.stash.appscode.com/gcs-repo created
```

Now, we are ready to backup our database into our desired backend.

## Backup

In this section, we are going to demonstrate `preBackup` hook and `postBackup` hook. We are going to make MySQL database read-only in  `preBackup` hook so that no write operation happens in the database during backup. Then, we are going to make the database writable in `postBackup` hook so that the application can write again into the database.

### PreBackup Hook

At first, we are going to set `super_read_only` flag `ON` in `preBackup` hook which will make the database read-only. However, we won't set this flag `OFF` in `postBackup` so that we can verify that the hook has been executed.

**Create BackupConfiguration:**

Below is the YAML of the `BackupConfiguration` CR with `preBackup` hook configured to make the database read-only before backup,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: backup-hook-demo
  namespace: demo
spec:
  schedule: "*/5 * * * *"
  task:
    name: mysql-backup-8.0.14
  repository:
    name: gcs-repo
  hooks:
    preBackup:
      exec:
        command:
          - /bin/sh
          - -c
          - mysql -u root --password=$MYSQL_ROOT_PASSWORD -e "SET GLOBAL super_read_only = ON;"
      containerName: mysql # KubeDB uses "mysql" name for MySQL database container. If you haven't used KubeDB, change this according to your setup.
  target:
    ref:
      apiVersion: appcatalog.appscode.com/v1alpha1
      kind: AppBinding
      name: sample-mysql
  retentionPolicy:
    name: keep-last-5
    keepLast: 5
    prune: true
```

Let's create the above `BackupConfiguration`,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/hooks/pre_backup_hook_demo.yaml
backupconfiguration.stash.appscode.com/backup-hook-demo created
```

**Verify CronJob:**

If everything goes well, Stash will create a CronJob with the schedule specified in `spec.schedule` field of the `BackupConfiguration` CR.

```bash
$ kubectl get cronjob -n demo
NAME                            SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
stash-backup-backup-hook-demo   */5 * * * *   False     0        <none>          74s
```

**Wait for BackupSession:**

The `stash-backup-backup-hook-demo` CronJob will trigger a backup on each scheduled slot by creating a `BackupSession` CR.

Wait for a schedule to appear. Run the following command to watch `BackupSession` CR,

```bash
$ kubectl get backupsession -n demo -w

NAME                          INVOKER-TYPE          INVOKER-NAME       PHASE       AGE
backup-hook-demo-1579179002   BackupConfiguration   backup-hook-demo   Running     10s
backup-hook-demo-1579179002   BackupConfiguration   backup-hook-demo   Running     52s
backup-hook-demo-1579179002   BackupConfiguration   backup-hook-demo   Succeeded   86s
```

Here, the phase `Succeeded` means that the backup process has been completed successfully.

**Verify Backup:**

Once a backup is completed, Stash will update the respective `Repository` CR to reflect the backup completion. Check that the repository `gcs-repo` has been updated by the following command,

```bash
$ kubectl get repository -n demo gcs-repo
NAME       INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
gcs-repo   true               1                75s                      55m
```

Here, `SNAPSHOT-COUNT` 1 indicates that one snapshot has been taken for the targeted database.

**Verify PreBackup Hook Executed:**

If the `preBackup` hook executes successfully, the database will be marked as read-only. In this situation, if we try to make a write operation into the database, it should reject the operation. However, the database should serve the read operations without any problem.

Let's verify that the database is read-only by trying to execute a write operation,

```bash
$ kubectl exec -it -n demo sample-mysql-0 -- mysql --user=$MYSQL_USER --password=$MYSQL_PASSWORD -e "CREATE DATABASE read-OnlyTest;"
mysql: [Warning] Using a password on the command line interface can be insecure.
ERROR 1290 (HY000) at line 1: The MySQL server is running with the --super-read-only option so it cannot execute this statement
command terminated with exit code 1
```

Here, the error message clearly states the database is now read-only. Let's try to execute a read operation.

```bash
$ kubectl exec -it -n demo sample-mysql-0 -- mysql --user=$MYSQL_USER --password=$MYSQL_PASSWORD -e "SELECT * FROM companyRecord.employee;"
mysql: [Warning] Using a password on the command line interface can be insecure.
+----+----------+--------+
| id | name     | salary |
+----+----------+--------+
|  1 | John Doe |   5000 |
+----+----------+--------+
```

So, we can see that the database can serve read-only queries without any problem.

### PostBackup Hook

Now, let's update the `BackupConfiguration` CR and add a `postBackup` hook that set `super_read_only` flag to `OFF`. So, the database should be writable again from the next backup.

**Update BackupConfiguration:**

Below is the YAML for the updated `BackupConfiguration` CR with `postBackup` hook.

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: backup-hook-demo
  namespace: demo
spec:
  schedule: "*/5 * * * *"
  task:
    name: mysql-backup-8.0.14
  repository:
    name: gcs-repo
  hooks:
    preBackup:
      exec:
        command:
          - /bin/sh
          - -c
          - mysql -u root --password=$MYSQL_ROOT_PASSWORD -e "SET GLOBAL super_read_only = ON;"
      containerName: mysql # KubeDB uses "mysql" name for MySQL database container. If you haven't used KubeDB, change this according to your setup.
    postBackup:
      exec:
        command:
          - /bin/sh
          - -c
          - mysql -u root --password=$MYSQL_ROOT_PASSWORD -e "SET GLOBAL super_read_only = OFF;"
      containerName: mysql
  target:
    ref:
      apiVersion: appcatalog.appscode.com/v1alpha1
      kind: AppBinding
      name: sample-mysql
  retentionPolicy:
    name: keep-last-5
    keepLast: 5
    prune: true
```

Let's apply the update,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/hooks/post_backup_hook_demo.yaml
backupconfiguration.stash.appscode.com/backup-hook-demo configured
```

**Wait for Next BackupSession:**

Now, wait for the next backup slot,

```bash
$ kubectl get backupsession -n demo -w

NAME                          INVOKER-TYPE          INVOKER-NAME       PHASE       AGE
backup-hook-demo-1579179002   BackupConfiguration   backup-hook-demo   Succeeded   7m8s
backup-hook-demo-1579179905   BackupConfiguration   backup-hook-demo   Running     12s
backup-hook-demo-1579179905   BackupConfiguration   backup-hook-demo   Running     8s
backup-hook-demo-1579179905   BackupConfiguration   backup-hook-demo   Succeeded   63s
```

**Verify PostBackup Hook Executed:**

If the `postBackup` hook has been executed successfully, the database should be writable again. Let's try to execute a write operation to verify that the database writable,

```bash
$ kubectl exec -it -n demo sample-mysql-0 -- mysql --user=$MYSQL_USER --password=$MYSQL_PASSWORD -e "CREATE DATABASE postBackupHookTest;"
mysql: [Warning] Using a password on the command line interface can be insecure.
```

Verify the test database has been created successfully,

```bash
$ kubectl exec -it -n demo sample-mysql-0 -- mysql --user=$MYSQL_USER --password=CWg2hru8b0Yu7dzS -e "SHOW DATABASES;"

mysql: [Warning] Using a password on the command line interface can be insecure.
+--------------------+
| Database           |
+--------------------+
| companyRecord      |
| information_schema |
| mysql              |
| performance_schema |
| postBackupHookTest |
| sys                |
+--------------------+
```

So, we can see the database is writable again after the backup.

## Restore

In this section, we are going to demonstrate `preRestore` and `postRestore` hooks. Here, we are going to delete corrupted data in `preRestore` hook and apply some migration on the database in `postRestore` hook.

**Pause Backup:**

At first, let stop the backup so that no new backup happens during the restore process. Let's set `spec.paused` section of `BackupConfiguration` to `true` which will stop taking further scheduled backup.

```bash
$ kubectl patch backupconfiguration -n demo backup-hook-demo --type="merge" --patch='{"spec": {"paused": true}}'
backupconfiguration.stash.appscode.com/backup-hook-demo patched
```

It should suspend the respective CronJob which is responsible for triggering backup at a scheduled slot. Let's verify that the CronJob has been suspended.

```bash
$ kubectl get cronjob -n demo
NAME                            SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
stash-backup-backup-hook-demo   */5 * * * *   True      0        5m13s           29m
```

**Simulate Disaster Scenario:**

Now, let's simulate a disaster scenario. Here, we are going to delete the `companyRecord` database before restoring so that we can verify that the data has been restored from backup.

```bash
$ kubectl exec -it -n demo sample-mysql-0 -- mysql --user=$MYSQL_USER --password=$MYSQL_PASSWORD -e "DROP DATABASE companyRecord;"
mysql: [Warning] Using a password on the command line interface can be insecure.
```

Verify that the database has been deleted,

```bash
$ kubectl exec -it -n demo sample-mysql-0 -- mysql --user=$MYSQL_USER --password=$MYSQL_PASSWORD -e "SHOW DATABASES;"
mysql: [Warning] Using a password on the command line interface can be insecure.
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| postBackupHookTest |
| sys                |
+--------------------+
```

So, we can see from the above output that the database `companyRecord` has been deleted from the MySQL server.

### PreRestore Hook

Here, we are going to configure `preRestore` hook to delete the corrupted database. Stash will remove the corrupted database first, then it will restore the database from the backup.

**Create RestoreSession:**

Below is the YAML for `RestoreSession` with `preRestore` hook configured to drop the `companyRecord` database before restoring from backup.

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: pre-restore-hook-demo
  namespace: demo
spec:
  task:
    name: mysql-restore-8.0.14
  repository:
    name: gcs-repo
  hooks:
    preRestore:
      exec:
        command:
          - /bin/sh
          - -c
          - mysql -u root --password=$MYSQL_ROOT_PASSWORD -e "DROP DATABASE companyRecord;"
      containerName: mysql
  target:
    ref:
      apiVersion: appcatalog.appscode.com/v1alpha1
      kind: AppBinding
      name: sample-mysql
    rules:
      - snapshots: [latest]
```

Let's create the above `RestoreSession`,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/hooks/pre_restore_hook_demo.yaml
restoresession.stash.appscode.com/pre-restore-hook-demo created
```

**Wait for Restore to Complete:**

Now, wait for the restore process to complete,

```bash
$ kubectl get restoresession -n demo -w
NAME                    REPOSITORY   PHASE     AGE
pre-restore-hook-demo   gcs-repo     Running   10s
pre-restore-hook-demo   gcs-repo     Running   42s
pre-restore-hook-demo   gcs-repo     Succeeded   42s
```

Here, `RestoreSession` phase `Succeeded` means the restore process has been completed successfully.

**Verify Restored Data:**

Verify that the data has been restored successfully,

```bash
$ kubectl exec -it -n demo sample-mysql-0 -- mysql --user=$MYSQL_USER --password=$MYSQL_PASSWORD -e "SELECT * FROM companyRecord.employee;"
mysql: [Warning] Using a password on the command line interface can be insecure.
+----+----------+--------+
| id | name     | salary |
+----+----------+--------+
|  1 | John Doe |   5000 |
+----+----------+--------+
```

So, we can see that the data we had deleted from the `employee` table has been restored.

### PostRestore Hook

Now, let's consider that you want to perform some migration on the database during the restore process. You want to rename the `employee` table into `salaryRecord` as it holds the employee's salary information. You can configure a `postRestore` hook to perform the task automatically.

**Drop Old Database:**

Let's delete the old database `companyRecord` before restoring so that we can verify that the data has been restored from backup.

```bash
$ kubectl exec -it -n demo sample-mysql-0 -- mysql --user=$MYSQL_USER --password=$MYSQL_PASSWORD -e "DROP DATABASE companyRecord;"
mysql: [Warning] Using a password on the command line interface can be insecure.
```

Verify that the database has been deleted,

```bash
$ kubectl exec -it -n demo sample-mysql-0 -- mysql --user=$MYSQL_USER --password=$MYSQL_PASSWORD -e "SHOW DATABASES;"
mysql: [Warning] Using a password on the command line interface can be insecure.
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| postBackupHookTest |
| sys                |
+--------------------+
```

**Create RestoreSession:**

Below is the YAML of the `RestoreSession` with `postRestore` hook configured to rename the `employee` table into `salaryRecord`.

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: post-restore-hook-demo
  namespace: demo
spec:
  task:
    name: mysql-restore-8.0.14
  repository:
    name: gcs-repo
  hooks:
    postRestore:
      exec:
        command:
          - /bin/sh
          - -c
          - mysql -u root --password=$MYSQL_ROOT_PASSWORD -e "RENAME TABLE companyRecord.employee TO companyRecord.salaryRecord;"
      containerName: mysql
  target:
    ref:
      apiVersion: appcatalog.appscode.com/v1alpha1
      kind: AppBinding
      name: sample-mysql
    rules:
      - snapshots: [latest]
```

Let's create the above `RestoreSession`,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/examples/guides/hooks/post_restore_hook_demo.yaml
restoresession.stash.appscode.com/post-restore-hook-demo created
```

**Wait for Restore process to Complete:**

Now, wait for the restore process to complete,

```bash
$ kubectl get restoresession -n demo post-restore-hook-demo -w
NAME                     REPOSITORY   PHASE     AGE
post-restore-hook-demo   gcs-repo     Running   12s
post-restore-hook-demo   gcs-repo     Running   29s
post-restore-hook-demo   gcs-repo     Succeeded   29s
```

**Verify Restored Data:**

Verify that the `companyRecord` database has been restored and the `employee` table has been renamed to `salaryRecord`.

```bash
$ kubectl exec -it -n demo sample-mysql-0 -- mysql --user=$MYSQL_USER --password=$MYSQL_PASSWORD -e "SHOW TABLES IN companyRecord;"
mysql: [Warning] Using a password on the command line interface can be insecure.
+-------------------------+
| Tables_in_companyRecord |
+-------------------------+
| salaryRecord            |
+-------------------------+
```

Let's check `salaryRecord` table contains the original data of the `employee` table,

```bash
$ kubectl exec -it -n demo sample-mysql-0 -- mysql --user=$MYSQL_USER --password=$MYSQL_PASSWORD -e "SELECT * FROM companyRecord.salaryRecord;"
mysql: [Warning] Using a password on the command line interface can be insecure.
+----+----------+--------+
| id | name     | salary |
+----+----------+--------+
|  1 | John Doe |   5000 |
+----+----------+--------+
```

So, we can see that the `postRestore` hook successfully performed migration on the restored database.

## Cleanup

To cleanup the Kubernetes resources created by this tutorial, run:

```bash
kubectl delete -n demo restoresession pre-restore-hook-demo post-restore-hook-demo
kubectl delete -n demo backupconfiguration backup-hook-demo
kubectl delete -n demo repository gcs-repo
kubectl delete -n demo secret gcs-secret
kubectl delete -n demo mysql sample-mysql
```
