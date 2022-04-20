---
title: Cross-Namespace-Target Backup and Restore | Stash
description: A guide on how to use backup and restore targetting workloads of different namespaces using Stash.
menu:
  docs_{{ .version }}:
    identifier: use-cases-cross-namespace-target
    name: Backup and Restore Cross-Namespace Target
    parent: use-cases
    weight: 70
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: guides
---

# Backup and Restore Cross-Namespace Target

This guide will show you how to take backup and restore targetting workloads of different namespaces using Stash.

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [kind](https://kind.sigs.k8s.io/docs/user/quick-start/).

- Install `Stash` in your cluster following the steps [here](/docs/setup/README.md).

- You should be familiar with the following `Stash` concepts:
  - [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md)
  - [BackupSession](/docs/concepts/crds/backupsession.md)
  - [RestoreSession](/docs/concepts/crds/restoresession.md)
  - [Repository](/docs/concepts/crds/repository.md)

To demonstrate the cross-namespace capability, we are going to keep our target workloads in the `prod` when taking backup and in the `staging` namespace when restoring. We will perform backup and restore from the `dev` namespace.

Let's create the above-mentioned namespaces,

```bash
$ kubectl create ns prod
namespace/prod created

$ kubectl create ns dev
namespace/dev created

$ kubectl create ns staging
namespace/staging created
```

>**Note:** YAML files used in this tutorial can be found [here](https://github.com/stashed/docs/guides/use-cases/cross-namespace-backup/examples).

## Backup 

This section will demonstrate taking backup of a MySQL instance of `prod` namespace from the `dev` namespace.

### Deploy Sample MySQL Database

We are going to use KubeDB for deploying a sample MySQL Database. Let's deploy the following sample MySQL database in prod namespace,

```yaml
apiVersion: kubedb.com/v1alpha2
kind: MySQL
metadata:
  name: sample-mysql
  namespace: dev
spec:
  version: "8.0.27"
  replicas: 1
  storageType: Durable
  storage:
    storageClassName: "standard"
    accessModes:
    - ReadWriteOnce
    resources:
      requests:
        storage: 1Gi
  terminationPolicy: WipeOut
```

To create the above `MySQL` CRD,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/use-cases/cross-namespace-target/examples/mysql.yaml
mysql.kubedb.com/sample-mysql created
```

KubeDB will deploy a MySQL database according to the above specification. It will also create the necessary Secrets and Services to access the database.

Let's check if the database is ready to use,

```bash
$ kubectl get my -n prod sample-mysql
NAME           VERSION   STATUS   AGE
sample-mysql   8.0.27    Ready    3m32s
```
The database is `Ready`. 

**Verify AppBinding:**

Verify that the AppBinding has been created successfully using the following command,

```bash
$ kubectl get appbindings -n prod
NAME           TYPE               VERSION   AGE
sample-mysql   kubedb.com/mysql   8.0.27    11m
```
Stash uses the AppBinding CRD to connect with the target database. 

**Insert Sample Data:**

Now, we are going to exec into the database pod and create some sample data. At first, find out the database Pod using the following command,

```bash
$ kubectl get pods -n prod --selector="app.kubernetes.io/instance=sample-mysql"
NAME             READY   STATUS    RESTARTS   AGE
sample-mysql-0   1/1     Running   0          33m
```

And copy the user name and password of the `root` user to access into `mysql` shell.

```bash
$ kubectl get secret -n prod  sample-mysql-auth -o jsonpath='{.data.username}'| base64 -d
root⏎

$ kubectl get secret -n prod  sample-mysql-auth -o jsonpath='{.data.password}'| base64 -d
16yGTTyhkROrQhH9⏎
```

Now, let's exec into the Pod to enter into `mysql` shell and create a database and a table,

```bash
$ kubectl exec -it -n prod sample-mysql-0 -- mysql --user=root --password="16yGTTyhkROrQhH9"
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 10
Server version: 8.0.14 MySQL Community Server - GPL

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> CREATE DATABASE playground;
Query OK, 1 row affected (0.01 sec)

mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| playground         |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> CREATE TABLE playground.equipment ( id INT NOT NULL AUTO_INCREMENT, type VARCHAR(50), quant INT, color VARCHAR(25), PRIMARY KEY(id));
Query OK, 0 rows affected (0.01 sec)

mysql> SHOW TABLES IN playground;
+----------------------+
| Tables_in_playground |
+----------------------+
| equipment            |
+----------------------+
1 row in set (0.01 sec)

mysql> INSERT INTO playground.equipment (type, quant, color) VALUES ("slide", 2, "blue");
Query OK, 1 row affected (0.01 sec)

mysql> SELECT * FROM playground.equipment;
+----+-------+-------+-------+
| id | type  | quant | color |
+----+-------+-------+-------+
|  1 | slide |     2 | blue  |
+----+-------+-------+-------+
1 row in set (0.00 sec)

mysql> exit
Bye
```

Now, we are ready to backup the database.

### Prepare Backend

We are going to store our backed-up data into a GCS bucket. We have to create a Secret with the necessary credentials and a Repository CRD to use this backend. 

If you want to use a different backend, please read the doc [here](/docs/guides/backends/overview.md).

> For GCS backend, if the bucket does not exist, Stash needs `Storage Object Admin` role permissions to create the bucket. For more details, please check the following [guide](/docs/guides/backends/gcs.md).

**Create Secret:**

Let's create a secret called `gcs-secret` in `dev` namespace with access credentials to our desired GCS bucket,

```bash
$ echo -n 'changeit' > RESTIC_PASSWORD
$ echo -n '<your-project-id>' > GOOGLE_PROJECT_ID
$ cat /path/to/downloaded-sa-key.json > GOOGLE_SERVICE_ACCOUNT_JSON_KEY
$ kubectl create secret generic -n dev gcs-secret \
    --from-file=./RESTIC_PASSWORD \
    --from-file=./GOOGLE_PROJECT_ID \
    --from-file=./GOOGLE_SERVICE_ACCOUNT_JSON_KEY
secret/gcs-secret created
```

**Create Repository:**

Now, create a Repository using this secret. Below is the YAML of Repository object we are going to create, 

```yaml
apiVersion: stash.appscode.com/v1alpha1
kind: Repository
metadata:
  name: gcs-repo
  namespace: dev
spec:
  backend:
    gcs:
      bucket: stash-testing
      prefix: /cross-namespace-target/data/sample-mysql
    storageSecretName: gcs-secret
```

Let's create the Repository we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/use-cases/cross-namespace-target/examples/repository.yaml
repository.stash.appscode.com/gcs-repo created
```
Now, we are ready to backup our sample data into this backend.

### Configure Backup

We are going to create a `BackupConfiguration` object in the `dev` namespace targeting the `sample-mysql` database of `prod` namespace. BackupConfiguration object does not have required RBAC permissions to take backup of target from another namespace. We are going to grant the necessary RBAC permissions through a ServiceAccount object. Let's create the RBAC resources first.

**Create ServiceAccount:**

We are going to create a ServiceAccount in the `dev` namespace. This ServiceAccount will refer to the granted permissions of `prod` namespace. Here is the YAML of the ServiceAccount,

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: mysql-sa
  namespace: dev
```

Let's create the ServiceAccount,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/use-cases/cross-namespace-target/examples/serviceaccount.yaml
serviceaccount/mysql-sa created
```

**Create Role**

We are going to create a Role in `prod` namespace with the necessary permissions to perform the backup. Here is the YAML of the Role,

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: mysql-role
  namespace: prod
rules:
- apiGroups: [""]
  resources: ["secrets", "endpoints", "pods"]
  verbs: ["get"]
- apiGroups: [""]
  resources: ["pods/exec"]
  verbs: ["get","create"]
- apiGroups: ["appcatalog.appscode.com"]
  resources: ["appbindings"]
  verbs: ["get"]
```

Let's create the Role we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/use-cases/cross-namespace-target/examples/role.yaml
role.rbac.authorization.k8s.io/mysql-role created
```

**Create RoleBinding:**

Now, we are going to create a RoleBinding in the `prod` namespace. This RoleBinding will refer to the `mysql-role` as roleref and `mysql-sa` as a subject which we have created earlier. Here is the YAML of the RoleBinding we are going to create,

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: mysql-rolebinding
  namespace: prod
subjects:
- kind: ServiceAccount
  name: mysql-sa
  namespace: dev
roleRef:
  kind: Role
  name: mysql-role
  apiGroup: rbac.authorization.k8s.io
```

Let's create the above RoleBinding,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/use-cases/cross-namespace-target/examples/rolebinding.yaml
rolebinding.rbac.authorization.k8s.io/mysql-rolebinding created
```

**Create BackupConfiguration:**

Now, we are going to create the BackupConfiguration to backup our MySQL target of `prod` namespace. Below is the YAML of the `BackupConfiguration` object,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: sample-mysql-backup
  namespace: dev
spec:
  schedule: "*/5 * * * *"
  repository:
    name: gcs-repo
  target:
    ref:
      apiVersion: appcatalog.appscode.com/v1alpha1
      kind: AppBinding
      name: sample-mysql
      namespace: prod
  runtimeSettings:
    pod:
      serviceAccountName: mysql-sa
  retentionPolicy:
    name: keep-last-5
    keepLast: 5
    prune: true
```
Note that, we have mentioned our ServiceAccount's name we have created earlier in the `spec.runtimeSettings.pod.serviceAccountName` section of the BackupConfiguration object. We are granting necesssary permissions to perform backup from `dev` namespace targetting an object of `prod` namespace through this ServiceAccount. 

Let's create the `BackupConfiguration` object we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/use-cases/cross-namespace-target/examples/backupconfiguration.yaml
backupconfiguration.stash.appscode.com/sample-mysql-backup

**Verify BackupConfiguration Ready:**

If everything goes well, the phase of the BackupConfiguration should be `Ready`. Let's check the BackupConfiguration Phase,

```bash
❯ kubectl get backupconfiguration -n prod
NAME                   TASK   SCHEDULE      PAUSED   PHASE   AGE
sample-mysql-backup           */5 * * * *            Ready   13s
```


### Verify Backup

The `sample-mysql-backup` BackupConfiguration will create a CronJob in `dev` namespace and that will trigger a backup on each scheduled slot by creating a `BackupSession` object.

Wait for the next schedule for the backup. Run the following command to watch `BackupSession` object,

```bash
$ kubectl get backupsession -n dev -w

NAME                             INVOKER-TYPE          INVOKER-NAME          PHASE       DURATION   AGE
sample-mysql-backup-1650452100   BackupConfiguration   sample-mysql-backup   Running                0s
sample-mysql-backup-1650452100   BackupConfiguration   sample-mysql-backup   Running                16s
sample-mysql-backup-1650452100   BackupConfiguration   sample-mysql-backup   Running                32s
sample-mysql-backup-1650452100   BackupConfiguration   sample-mysql-backup   Succeeded   33s        32s
```

We can see from the above that the Phase of the  BackupSession  is Succeeded. It indicates that Stash has successfully taken backup of our target. 

## Restore

In this section, we are going to restore the database  in the `staging` namespace from the backup we have taken in the previous section. We are going to deploy a new database and initialize it from the backup.

**Stop Taking Backup of the Old Database:**

At first, let's stop taking any further backup of the old database so that no backup is taken during restore process.

Let's pause the `sample-mysql-backup` BackupConfiguration,

```console
$ kubectl patch backupconfiguration -n dev sample-mysql-backup --type="merge" --patch='{"spec": {"paused": true}}'
backupconfiguration.stash.appscode.com/sample-mysql-backup patched
```

Now, wait for a moment. Stash will pause the BackupConfiguration. Verify that the BackupConfiguration  has been paused,

```console
$ kubectl get backupconfiguration -n dev sample-mysql-backup
NAME                 TASK                  SCHEDULE      PAUSED   PHASE   AGE
sample-mysql-backup  mysql-backup-8.0.21   */5 * * * *   true     Ready   26m
```

Notice the `PAUSED` column. Value `true` for this field means that the BackupConfiguration has been paused.

### Deploy Recovery MySQL Database

Now, we are going to deploy a new database in the `staging` namespace similarly as we have deployed the `sample-mysql` database. However, this time we are going to specify `spec.init.waitForInitialRestore: true` which will tell KubeDB to wait until the first restore to complete before marking this database as ready to use.

Below is the YAML for `MySQL` CRD we are going deploy to initialize from backup,

```yaml
apiVersion: kubedb.com/v1alpha2
kind: MySQL
metadata:
  name: mysql-recovery
  namespace: staging
spec:
  version: "8.0.27"
  replicas: 1
  storageType: Durable
  storage:
    storageClassName: "standard"
    accessModes:
    - ReadWriteOnce
    resources:
      requests:
        storage: 1Gi
  init:
    waitForInitialRestore: true
  terminationPolicy: WipeOut
```

Let's create the above database,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/use-cases/cross-namespace-target/examples/mysql_recovery.yaml
mysql.kubedb.com/mysql-recovery created
```

If you check the database status, you will see it is stuck in `Provisioning` state.

```bash
$ kubectl get my -n staging mysql-recovery
NAME             VERSION   STATUS         AGE
mysql-recovery   8.0.27    Provisioning   75s
```

### Configure Restore

We are going to create a `RestoreSession` object in the `dev` namespace targeting the `mysql-recovery` database of `staging` namespace. Similar to BackupConfiguration, we need to grant the necessary RBAC permissions through a ServiceAccount object to the RestoreSession for performing restoration in a different namespace. Let's create the RBAC resources first.


**Create Role**

We are going to create a Role in `prod` namespace with the necessary permissions to perform the restore. Here is the YAML of the Role,

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: restore-mysql-role
  namespace: prod
rules:
- apiGroups: [""]
  resources: ["secrets", "endpoints", "pods"]
  verbs: ["get"]
- apiGroups: [""]
  resources: ["pods/exec"]
  verbs: ["get","create"]
- apiGroups: ["appcatalog.appscode.com"]
  resources: ["appbindings"]
  verbs: ["get"]
```

Let's create the Role we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/use-cases/cross-namespace-target/examples/role.yaml
role.rbac.authorization.k8s.io/mysql-role created
```

**Create RoleBinding:**

We are going to create a RoleBinding in the `staging` namespace. This RoleBinding will refer to the `mysql-role` as roleref and `mysql-sa` as a subject which we have created earlier. Here is the YAML of the RoleBinding we are going to create,

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: mysql-rolebinding
  namespace: prod
subjects:
- kind: ServiceAccount
  name: mysql-sa
  namespace: dev
roleRef:
  kind: Role
  name: mysql-role
  apiGroup: rbac.authorization.k8s.io
```

Let's create the above RoleBinding,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/use-cases/cross-namespace-target/examples/rolebinding.yaml
rolebinding.rbac.authorization.k8s.io/mysql-rolebinding created
```

**Create RestoreSession:**

Now, we need to create a RestoreSession CRD pointing to the AppBinding for this restored database.

Using the following command, check that another AppBinding object has been created for the `restored-mysql` object,

```bash
$ kubectl get appbindings -n demo restored-mysql
NAME             AGE
restored-mysql   6m6s
```

> If you are not using KubeDB to deploy database, create the AppBinding manually.

Below is the contents of YAML file of the RestoreSession CRD that we are going to create to restore backed up data into the newly created database provisioned by MySQL CRD named `restored-mysql`.

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: sample-mysql-restore
  namespace: demo
spec:
  task:
    name: mysql-restore-8.0.21
  repository:
    name: gcs-repo
  target:
    ref:
      apiVersion: appcatalog.appscode.com/v1alpha1
      kind: AppBinding
      name: restored-mysql
  rules:
    - snapshots: [latest]
```

Here,

- `.spec.task.name` specifies the name of the Task CRD that specifies the necessary Functions and their execution order to restore a MySQL database.
- `.spec.repository.name` specifies the Repository CRD that holds the backend information where our backed up data has been stored.
- `.spec.target.ref` refers to the newly created AppBinding object for the `restored-mysql` MySQL object.
- `.spec.rules` specifies that we are restoring data from the latest backup snapshot of the database.

Let's create the RestoreSession CRD object we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/addons/mysql/standalone/examples/restoresession.yaml
restoresession.stash.appscode.com/sample-mysql-restore created
```

Once, you have created the RestoreSession object, Stash will create a restore Job. We can watch the phase of the RestoreSession object to check whether the restore process has succeeded or not.

Run the following command to watch the phase of the RestoreSession object,

```bash
$ watch -n 1 kubectl get restoresession -n demo restore-sample-mysql

Every 1.0s: kubectl get restoresession -n demo  restore-sample-mysql    workstation: Fri Sep 27 11:18:51 2019
NAMESPACE   NAME                   REPOSITORY-NAME   PHASE       AGE
demo        restore-sample-mysql   gcs-repo          Succeeded   59s
```

Here, we can see from the output of the above command that the restore process succeeded.

**Verify Restored Data:**

In this section, we are going to verify whether the desired data has been restored successfully. We are going to connect to the database server and check whether the database and the table we created earlier in the original database are restored.

At first, check if the database has gone into **`Running`** state by the following command,

```bash
$ kubectl get my -n demo restored-mysql
NAME             VERSION   STATUS    AGE
restored-mysql   8.0.14    Ready     34m
```

Now, find out the database Pod by the following command,

```bash
$ kubectl get pods -n demo --selector="app.kubernetes.io/instance=restored-mysql"
NAME               READY   STATUS    RESTARTS   AGE
restored-mysql-0   1/1     Running   0          39m
```

And then copy the user name and password of the `root` user to access into `mysql` shell.

> Notice: We used the same Secret for the `restored-mysql` object. So, we will use the same commands as before.

```bash
$ kubectl get secret -n demo  sample-mysql-auth -o jsonpath='{.data.username}'| base64 -d
root⏎

$ kubectl get secret -n demo  sample-mysql-auth -o jsonpath='{.data.password}'| base64 -d
5HEqoozyjgaMO97N⏎
```

Now, let's exec into the Pod to enter into `mysql` shell and create a database and a table,

```bash
$ kubectl exec -it -n demo restored-mysql-0 -- mysql --user=root --password=5HEqoozyjgaMO97N
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.14 MySQL Community Server - GPL

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| playground         |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> SHOW TABLES IN playground;
+----------------------+
| Tables_in_playground |
+----------------------+
| equipment            |
+----------------------+
1 row in set (0.00 sec)

mysql> SELECT * FROM playground.equipment;
+----+-------+-------+-------+
| id | type  | quant | color |
+----+-------+-------+-------+
|  1 | slide |     2 | blue  |
+----+-------+-------+-------+
1 row in set (0.00 sec)

mysql> exit
Bye
```

So, from the above output, we can see that the `playground` database and the `equipment` table we created earlier in the original database and now, they are restored successfully.

## Cleanup

To cleanup the Kubernetes resources created by this tutorial, run:

```bash
kubectl delete backupconfiguration -n demo sample-mysql-backup
kubectl delete restoresession -n demo restore-sample-mysql
kubectl delete repository -n demo gcs-repo
kubectl delete my -n demo restored-mysql
kubectl delete my -n demo sample-mysql
```
