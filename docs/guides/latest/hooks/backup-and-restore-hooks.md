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

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [kind](https://kind.sigs.k8s.io/docs/user/quick-start/).
- Install Stash in your cluster following the steps [here](/docs/setup/install.md).
- Install MySQL addon for Stash following the steps [here](/docs/addons/mysql/setup/install.md).
- Install [KubeDB](https://kubedb.com) in your cluster following the steps [here](https://kubedb.com/docs/setup/install/). This step is optional. You can deploy your database using any method you want. We are using KubeDB because KubeDB simplifies many of the difficult or tedious management tasks of running a production grade databases on private and public clouds.
- If you are not familiar with how Stash backup and restore MySQL databases, please check the following guide [here](/docs/addons/mysql/overview.md).

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

## Prepare Database

**Deploy Database:**

```yaml
apiVersion: kubedb.com/v1alpha1
kind: MySQL
metadata:
  name: sample-mysql
  namespace: demo
spec:
  version: "8.0.14"
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

```console
$ kubectl apply -f ./docs/examples/guides/latest/hooks/sample-mysql.yaml
mysql.kubedb.com/sample-mysql created
```

Wait for the database to be ready,

```console
$ kubectl get mysql -n demo
NAME           VERSION   STATUS    AGE
sample-mysql   8.0.14    Running   2m7s
```

**Verify Database Secret:**

```console
$ kubectl get secret -n demo -l=kubedb.com/name=sample-mysql
NAME                TYPE     DATA   AGE
sample-mysql-auth   Opaque   2      5m7s
```

**Verify AppBinding:**

```console
$ kubectl get appbindings -n demo -l=kubedb.com/name=sample-mysql
NAME           TYPE               VERSION   AGE
sample-mysql   kubedb.com/mysql   8.0.14    66s
```

```console
kubectl get appbindings sample-mysql -n demo -o yaml
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
    kubedb.com/name: sample-mysql
  name: sample-mysql
  namespace: demo
  ownerReferences:
  - apiVersion: kubedb.com/v1alpha1
    blockOwnerDeletion: true
    controller: true
    kind: MySQL
    name: sample-mysql
    uid: 7e522f69-bdc4-4272-8463-bb4ab8ec2b86
  resourceVersion: "25860"
  selfLink: /apis/appcatalog.appscode.com/v1alpha1/namespaces/demo/appbindings/sample-mysql
  uid: 3b8a031a-bd60-42b3-9560-34b84e873624
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

Get database pod

```console
$ kubectl get pods -n demo --selector="kubedb.com/name=sample-mysql"
NAME             READY   STATUS    RESTARTS   AGE
sample-mysql-0   1/1     Running   0          6m50s
```

Get credentials

```console
$ export MYSQL_USER=$(kubectl get secret -n demo  sample-mysql-auth -o jsonpath='{.data.username}'| base64 -d)

$ echo $MYSQL_USER
root

$ export MYSQL_PASSWORD=$(kubectl get secret -n demo  sample-mysql-auth -o jsonpath='{.data.password}'| base64 -d)

$ echo $MYSQL_PASSWORD
CWg2hru8b0Yu7dzS
```

Exec into database pod

```console
$ kubectl exec -it -n demo sample-mysql-0 -- mysql --user=root --password=CWg2hru8b0Yu7dzS

mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 131
Server version: 8.0.14 MySQL Community Server - GPL

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> CREATE DATABASE companyRecord;
Query OK, 1 row affected (0.01 sec)

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

mysql> CREATE TABLE companyRecord.employee (id INT, name VARCHAR(50), salary INT, PRIMARY KEY(id));
Query OK, 0 rows affected (0.05 sec)

mysql> INSERT INTO companyRecord.employee (id, name, salary) VALUES (1, "John Doe", 5000);
Query OK, 1 row affected (0.01 sec)

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

We are going to store our backed up data into a GCS bucket. At first, we need to create a secret with GCS credentials then we need to create a `Repository` CRD. If you want to use a different backend, please read the respective backend configuration doc from [here](/docs/guides/latest/backends/overview.md).

**Create Storage Secret:**

Let's create a secret called `gcs-secret` with access credentials to our desired GCS bucket,

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

Now, crete a `Respository` using this secret. Below is the YAML of Repository CRD we are going to create,

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
$ kubectl apply -f ./docs/examples/guides/latest/hooks/repository.yaml
repository.stash.appscode.com/gcs-repo created
```

Now, we are ready to backup our database to our desired backend.

## Backup

### PreBackup Hook

**Create BackupConfiguration:**

```bash
kubectl apply -f ./docs/examples/guides/latest/hooks/pre_backup_hook_demo.yaml
backupconfiguration.stash.appscode.com/backup-hook-demo created
```

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

**Verify CronJob:**

```bash
$ kubectl get cronjob -n demo
NAME                            SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
stash-backup-backup-hook-demo   */5 * * * *   False     0        <none>          74s
```

**Wait for BackupSession:**

```console
Every 5.0s: kubectl get backupsession -n demo                                    workstation: Thu Jan 16 18:51:28 2020

NAME                          INVOKER-TYPE          INVOKER-NAME       PHASE       AGE
backup-hook-demo-1579179002   BackupConfiguration   backup-hook-demo   Succeeded   86s
```

**Verify Backup:**

```console
kubectl get repository -n demo gcs-repo
NAME       INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
gcs-repo   true               1                75s                      55m
```

**Verify PreBackup Hook Executed:**

```console
kubectl exec -it -n demo sample-mysql-0 -- mysql --user=root --password=CWg2hru8b0Yu7dzS -e "CREATE DATABASE readOnlyTest;"
mysql: [Warning] Using a password on the command line interface can be insecure.
ERROR 1290 (HY000) at line 1: The MySQL server is running with the --super-read-only option so it cannot execute this statement
command terminated with exit code 1
```

```console
$ kubectl exec -it -n demo sample-mysql-0 -- mysql --user=root --password=CWg2hru8b0Yu7dzS -e "SELECT * FROM companyRecord.employee;"
mysql: [Warning] Using a password on the command line interface can be insecure.
+----+----------+--------+
| id | name     | salary |
+----+----------+--------+
|  1 | John Doe |   5000 |
+----+----------+--------+
```

### PostBackup Hook

**Update BackupConfiguration:**

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
      containerName: mysql
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

```console
$ kubectl apply -f ./docs/examples/guides/latest/hooks/post_backup_hook_demo.yaml
backupconfiguration.stash.appscode.com/backup-hook-demo configured
```

**Wait for Next BackupSession:**

```console
$ watch -n 5 kubectl get backupsession -n demo

Every 5.0s: kubectl get backupsession -n demo                                    workstation: Thu Jan 16 19:06:08 2020

NAME                          INVOKER-TYPE          INVOKER-NAME       PHASE       AGE
backup-hook-demo-1579179905   BackupConfiguration   backup-hook-demo   Succeeded   63s
```

**Verify PostBackup Hook Executed:**

```console
$ kubectl exec -it -n demo sample-mysql-0 -- mysql --user=root --password=CWg2hru8b0Yu7dzS -e "CREATE DATABASE postBackupHookTest;"
mysql: [Warning] Using a password on the command line interface can be insecure.
```

```console
$ kubectl exec -it -n demo sample-mysql-0 -- mysql --user=root --password=CWg2hru8b0Yu7dzS -e "SHOW DATABASES;"

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

## Restore

**Pause Backup:**

```console
kubectl patch backupconfiguration -n demo backup-hook-demo --type="merge" --patch='{"spec": {"paused": true}}'
backupconfiguration.stash.appscode.com/backup-hook-demo patched
```

Verify

```console
kubectl get cronjob -n demo
NAME                            SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
stash-backup-backup-hook-demo   */5 * * * *   True      0        5m13s           29m
```

**Simulate Disaster Scenario:**

```console
kubectl exec -it -n demo sample-mysql-0 -- mysql --user=$MYSQL_USER --password=$MYSQL_PASSWORD -e "DELETE FROM companyRecord.employee;"
mysql: [Warning] Using a password on the command line interface can be insecure.
```

```console
kubectl exec -it -n demo sample-mysql-0 -- mysql --user=$MYSQL_USER --password=$MYSQL_PASSWORD -e "SELECT * FROM companyRecord.employee;"
mysql: [Warning] Using a password on the command line interface can be insecure.
```

### PreRestore Hook

**Create RestoreSession:**

```console
$ kubectl apply -f ./docs/examples/guides/latest/hooks/pre_restore_hook_demo.yaml
restoresession.stash.appscode.com/pre-restore-hook-demo created
```

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

**Wait for Restore to Complete:**

```console
$ kubectl get restoresession -n demo -w
NAME                    REPOSITORY   PHASE     AGE
pre-restore-hook-demo   gcs-repo     Running   10s
pre-restore-hook-demo   gcs-repo     Running   42s
pre-restore-hook-demo   gcs-repo     Succeeded   42s
```

**Verify Restored Data:**

```console
kubectl exec -it -n demo sample-mysql-0 -- mysql --user=$MYSQL_USER --password=$MYSQL_PASSWORD -e "SELECT * FROM companyRecord.employee;"
mysql: [Warning] Using a password on the command line interface can be insecure.
+----+----------+--------+
| id | name     | salary |
+----+----------+--------+
|  1 | John Doe |   5000 |
+----+----------+--------+
```

### PostRestore Hook

**Simulate Disaster:**

```console
kubectl exec -it -n demo sample-mysql-0 -- mysql --user=$MYSQL_USER --password=$MYSQL_PASSWORD -e "DROP DATABASE companyRecord;"
mysql: [Warning] Using a password on the command line interface can be insecure.
```

```console
kubectl exec -it -n demo sample-mysql-0 -- mysql --user=$MYSQL_USER --password=$MYSQL_PASSWORD -e "SHOW DATABASES;"
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

```console
$ kubectl apply -f ./docs/examples/guides/latest/hooks/post_restore_hook_demo.yaml
restoresession.stash.appscode.com/post-restore-hook-demo created
```

**Wait for Restore to Complete:**

```console
$ kubectl get restoresession -n demo post-restore-hook-demo -w
NAME                     REPOSITORY   PHASE     AGE
post-restore-hook-demo   gcs-repo     Running   12s
post-restore-hook-demo   gcs-repo     Running   29s
post-restore-hook-demo   gcs-repo     Succeeded   29s
```

**Verify Restored Data:**

```console
kubectl exec -it -n demo sample-mysql-0 -- mysql --user=$MYSQL_USER --password=$MYSQL_PASSWORD -e "SHOW TABLES IN companyRecord;"
mysql: [Warning] Using a password on the command line interface can be insecure.
+-------------------------+
| Tables_in_companyRecord |
+-------------------------+
| salaryRecord            |
+-------------------------+
```

```console
kubectl exec -it -n demo sample-mysql-0 -- mysql --user=$MYSQL_USER --password=$MYSQL_PASSWORD -e "SELECT * FROM companyRecord.salaryRecord;"
mysql: [Warning] Using a password on the command line interface can be insecure.
+----+----------+--------+
| id | name     | salary |
+----+----------+--------+
|  1 | John Doe |   5000 |
+----+----------+--------+
```

## Cleanup

```console
kubectl delete -n demo restoresession pre-restore-hook-demo post-restore-hook-demo
kubectl delete -n demo backupconfiguration backup-hook-demo
kubectl delete -n demo repository gcs-repo
kubectl delete -n demo secret gcs-secret
kubectl delete -n demo mysql sample-mysql
```
