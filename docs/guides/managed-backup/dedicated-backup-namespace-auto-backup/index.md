---
title: Auto Backup in Dedicated Backup Namespace | Stash
description: A guide on how to use dedicated backup namespace for automatic backup of targets of different namespaces using Stash.
menu:
  docs_{{ .version }}:
    identifier: managed-backup-dedicated-backup-namespace-auto-backup
    name: Auto Backup in Dedicated Backup Namespace
    parent: managed-backup
    weight: 30
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: guides
---

# Auto Backup in a Dedicated Namespace

This guide will show you how you can use a dedicated backup namespace for Auto Backup to keep your backup resources isolated from your workloads.

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [kind](https://kind.sigs.k8s.io/docs/user/quick-start/).

- Install `Stash` in your cluster following the steps [here](/docs/setup/README.md).

- You should be familiar with the following `Stash` concepts:
  - [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md)
  - [BackupSession](/docs/concepts/crds/backupsession.md)
  - [Repository](/docs/concepts/crds/repository.md)

Here, we are going to take backup from the `prod-1`, `prod-2` and prod-3` namespaces and manage the backup from `backup` namespace using Auto Backup.

Let's create the above-mentioned namespaces,

```bash
$ kubectl create ns prod-1
namespace/prod-1 created

$ kubectl create ns prod-2
namespace/prod-2 created

$ kubectl create ns prod-3
namespace/prod-3 created

$ kubectl create ns backup
namespace/backup created

```

>**Note:** YAML files used in this tutorial can be found [here](https://github.com/stashed/docs/guides/managed-backup/dedicated-backup-namespace-auto-backup/examples).

## Prepare Backup Blueprint

We are going to use [GCS Backend](/docs/guides/backends/gcs.md) to store the backed up data. You can use any supported backend you prefer. You just have to configure Storage Secret and `spec.backend` section of `BackupBlueprint` to match your backend. To learn which backends are supported by Stash and how to configure them, please visit [here](/docs/guides/backends/overview.md).

> For GCS backend, if the bucket does not exist, Stash needs `Storage Object Admin` role permissions to create the bucket. For more details, please check the following [guide](/docs/guides/backends/gcs.md).

**Create Storage Secret:**

At first, let's create a Storage Secret for the GCS backend in the `backup` namespace,

```bash
$ echo -n 'changeit' > RESTIC_PASSWORD
$ echo -n '<your-project-id>' > GOOGLE_PROJECT_ID
$ mv downloaded-sa-json.key GOOGLE_SERVICE_ACCOUNT_JSON_KEY
$ kubectl create secret generic -n backup gcs-secret \
    --from-file=./RESTIC_PASSWORD \
    --from-file=./GOOGLE_PROJECT_ID \
    --from-file=./GOOGLE_SERVICE_ACCOUNT_JSON_KEY
secret/gcs-secret created
```

Stash does not grant necessary RBAC permissions to the backup job for taking backup from a different namespace. In this case, we have to provide the RBAC permissions manually. This helps to prevent unauthorized namespaces from getting access to a database via Stash.

**Create ServiceAccount:**

At first, we are going to create a ServiceAccount in the `backup` namespace. This ServiceAccount will refer to the granted permissions for taking backup. Here is the YAML of the ServiceAccount,

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cross-namespace-target-reader
  namespace: backup
```

Let's create the ServiceAccount,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/managed-backup/dedicated-backup-namespace-auto-backup/examples/serviceaccount.yaml
serviceaccount/cross-namespace-target-reader created
```

**Create ClusterRole and ClusterRoleBinding**

We are going to create a ClusterRole and ClusterRoleBinding with the necessary permissions to perform the backup. Here are the YAMLs of the ClusterRole and ClusterRoleBinding,

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cross-namespace-target-clusterrole
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get","list"]
- apiGroups: ["appcatalog.appscode.com"]
  resources: ["appbindings"]
  verbs: ["get","list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cross-namespace-target-clusterrolebinding
subjects:
- kind: ServiceAccount
  name: cross-namespace-target-reader
  namespace: backup
roleRef:
  kind: ClusterRole
  name: cross-namespace-target-clusterrole
  apiGroup: rbac.authorization.k8s.io
```

Let's create the ClusterRole and ClusterRoleBinding we have shown above,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/managed-backup/dedicated-backup-namespace-auto-backup/examples/clusterrole_clusterrolebinding.yaml
clusterrole.rbac.authorization.k8s.io/cross-namespace-target-clusterrole created
clusterrolebinding.rbac.authorization.k8s.io/cross-namespace-target-clusterrolebinding created
```

The above RBAC permissions will allow ServiceAccounts to grant necessary backup job permissions from any namespace.

Alternatively, you can create Role and RoleBinding with the same permissions in case you want to restrict the ServiceAccounts to backup targets from only a definite namespace.

**Create BackupBlueprint:**

Now, we have to create a `BackupBlueprint` crd with a blueprint for `Repository` and `BackupConfiguration` object.

Below is the YAML of the `BackupBlueprint` object that we are going to create,

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupBlueprint
metadata:
  name: mysql-backup-blueprint
spec:
  backupNamespace: backup
  # ============== Blueprint for Repository ==========================
  backend:
    gcs:
      bucket: stash-testing
      prefix: stash-backup/${TARGET_NAMESPACE}/${TARGET_NAME}
    storageSecretName: gcs-secret
  # ============== Blueprint for BackupConfiguration =================
  schedule: "*/5 * * * *"
  retentionPolicy:
    name: 'keep-last-5'
    keepLast: 5
    prune: true
  runtimeSettings:
    pod:
      serviceAccountName: cross-namespace-target-reader
```

Note that we have used some variables (format: `${<variable name>}`) in `backend.gcs.prefix` field. Stash will substitute these variables with values from the respective target. Since the resolved prefix will be different for different target, the backed up data will be stored in different directory inside the bucket. To know which variable you can use in this `prefix` field, please visit [here](/docs/concepts/crds/backupblueprint.md#repository-blueprint).

Let's create the `BackupBlueprint` that we have shown above,

```bash
kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/managed-backup/dedicated-backup-namespace-auto-backup/examples/backupblueprint.yaml
backupblueprint.stash.appscode.com/mysql-backup-blueprint created
```
Now, automatic backup is configured for MySQL database. We just have to add some annotations to the targeted databases to enable periodic backup.

## Backup

In this section, we are going to backup three MySQL databases from the `prod-1`, `prod-2`, `prod-3`, namespaces.

### Deploy MySQL Databases

We are going to use KubeDB for deploying a sample MySQL Databases. Let's deploy three different MySQL databases in `prod-1`, `prod-2`, `prod-3` namespaces,

```yaml
apiVersion: kubedb.com/v1alpha2
kind: MySQL
metadata:
  name: sample-mysql-1
  namespace: prod-1
  annotations:
    stash.appscode.com/backup-blueprint: mysql-backup-blueprint
spec:
  version: "8.0.27"
  replicas: 1
  storageType: Durable
  storage:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 50Mi
  terminationPolicy: WipeOut
---
apiVersion: kubedb.com/v1alpha2
kind: MySQL
metadata:
  name: sample-mysql-2
  namespace: prod-2
  annotations:
    stash.appscode.com/backup-blueprint: mysql-backup-blueprint
spec:
  version: "8.0.27"
  replicas: 1
  storageType: Durable
  storage:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 50Mi
  terminationPolicy: WipeOut
---
apiVersion: kubedb.com/v1alpha2
kind: MySQL
metadata:
  name: sample-mysql-3
  namespace: prod-3
  annotations:
    stash.appscode.com/backup-blueprint: mysql-backup-blueprint
spec:
  version: "8.0.27"
  replicas: 1
  storageType: Durable
  storage:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 50Mi
  terminationPolicy: WipeOut

```

Notice the metadata.annotations field. We have specified to use mysql-backup-blueprint BackupBlueprint for creating `Repository` and `BackupConfiguration` for the MySQL databases. BackupBlueprint is a non-namespaced resource, so we just need to specify the name of the blueprint.

Let's create the above `MySQL` objects,

```bash
$ kubectl apply -f https://github.com/stashed/docs/raw/{{< param "info.version" >}}/docs/guides/managed-backup/dedicated-backup-namespace-auto-backup/examples/mysql.yaml
mysql.kubedb.com/sample-mysql-1 created
mysql.kubedb.com/sample-mysql-2 created
mysql.kubedb.com/sample-mysql-3 created
```

If everything goes well, Stash will create a Repository and a BackupConfiguration for each MySQL database with the name in the following format:
<target-namespace><target-kind>-<target-name>.

**Verify Repository:**

Verify that the Repositories has been created successfully by the following command,

```bash
$ kubectl get repository -n backup
NAME                        INTEGRITY   SIZE   SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
prod-1-app-sample-mysql-1                                                                31s
prod-2-app-sample-mysql-2                                                                31s
prod-3-app-sample-mysql-3                                                                30s
```

**Verify BackupConfiguratoin:**

If everything goes well, Stash should create a `BackupConfiguration` for each of the MySQL database and the phase of the `BackupConfiguration` should be `Ready`. Verify that the `BackupConfiguration`s have been created by the following command,

```bash
$ kubectl get backupconfiguration -n backup
NAME                        TASK   SCHEDULE      PAUSED   PHASE   AGE
prod-1-app-sample-mysql-1          */5 * * * *            Ready   11m
prod-2-app-sample-mysql-2          */5 * * * *            Ready   11m
prod-3-app-sample-mysql-3          */5 * * * *            Ready   11m
```

Let's check the YAML of one of the `BackupConfiguration`s,

```bash
$ kubectl get backupconfiguration -n backup prod-1-app-sample-mysql-1 -o yaml
```

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: prod-1-app-sample-mysql-1
  namespace: backup
  ...
spec:
  driver: Restic
  repository:
    name: prod-1-app-sample-mysql-1
    namespace: backup
  retentionPolicy:
    keepLast: 5
    name: keep-last-5
    prune: true
  runtimeSettings:
    pod:
      serviceAccountName: cross-namespace-target-reader
  schedule: '*/5 * * * *'
  target:
    ref:
      apiVersion: appcatalog.appscode.com/v1alpha1
      kind: AppBinding
      name: sample-mysql-1
      namespace: prod-1
  task: {}
  tempDir: {}
```

Notice that the `spec.target.ref` is pointing to the AppBinding of the database `sample-mysql-1` in the `prod-1` namespace.

**Wait for BackupSession:**

Run the following command to watch `BackupSession` crd:

```bash
$ watch -n 1 kubectl get backupsession -n backup
NAME                                   INVOKER-TYPE          INVOKER-NAME                PHASE       DURATION   AGE
prod-1-app-sample-mysql-1-1650885602   BackupConfiguration   prod-1-app-sample-mysql-1   Succeeded   45s        5m25s
prod-1-app-sample-mysql-1-1650885906   BackupConfiguration   prod-1-app-sample-mysql-1   Running                21s
prod-2-app-sample-mysql-2-1650885310   BackupConfiguration   prod-2-app-sample-mysql-2   Succeeded   40s        10m
prod-2-app-sample-mysql-2-1650885602   BackupConfiguration   prod-2-app-sample-mysql-2   Succeeded   41s        5m25s
prod-2-app-sample-mysql-2-1650885906   BackupConfiguration   prod-2-app-sample-mysql-2   Running                21s
prod-3-app-sample-mysql-3-1650885310   BackupConfiguration   prod-3-app-sample-mysql-3   Succeeded   38s        10m
prod-3-app-sample-mysql-3-1650885602   BackupConfiguration   prod-3-app-sample-mysql-3   Succeeded   41s        5m25s

```

>Note: Respective CronJob creates `BackupSession` crd with the following label `stash.appscode.com/backup-configuration=<BackupConfiguration crd name>`. We can use this label to watch only the `BackupSession` of our desired `BackupConfiguration`.ore

In this section, we are going to restore the database into the `staging` namespace from the backup we have taken in the previous section.

**Verify Backup:**

When backup session is completed, Stash will update the respective `Repository` to reflect the latest state of backed up data.

Run the following command to check if the snapshots are stored in the backend,

```bash
$ kubectl get repository -n backup 
NAME                        INTEGRITY   SIZE        SNAPSHOT-COUNT   LAST-SUCCESSFUL-BACKUP   AGE
prod-1-app-sample-mysql-1   true        4.174 MiB   5                2m50s                    50m
prod-2-app-sample-mysql-2   true        7.283 MiB   5                2m50s                    50m
prod-3-app-sample-mysql-3   true        4.701 MiB   5                2m50s                    50m
```

If we navigate to `stash-backup/prod-1/sample-mysql-1`, `stash-backup/prod-2/sample-mysql-2` and `stash-backup/prod-3/sample-mysql-3`  directories of our GCS bucket, we are going to see that the snapshots of the databases `sample-mysql-1`, `sample-mysql-2` and`sample-mysql-3` have been stored there respectively.

<figure align="center">
  <img alt="Backup data in GCS backend" src="/docs/guides/managed-backup/dedicated-backup-namespace-auto-backup/images/gcs-prod-1.png">
  <figcaption align="center">Fig: Backup data of `sample-mysql-1` in GCS backend</figcaption>
</figure>

<figure align="center">
  <img alt="Backup data in GCS backend" src="/docs/guides/managed-backup/dedicated-backup-namespace-auto-backup/images/gcs-prod-2.png">
  <figcaption align="center">Fig: Backup data of `sample-mysql-2` in GCS backend</figcaption>
</figure>

<figure align="center">
  <img alt="Backup data in GCS backend" src="/docs/guides/managed-backup/dedicated-backup-namespace-auto-backup/images/gcs-prod-3.png">
  <figcaption align="center">Fig: Backup data of `sample-mysql-3` in GCS backend</figcaption>
</figure>


## Cleanup

To cleanup the Kubernetes resources created by this tutorial, run:

```bash
$ kubectl delete backupblueprint mysql-backup-blueprint
backupblueprint.stash.appscode.com "mysql-backup-blueprint" deleted
$ kubectl delete backupconfiguration -n backup --all
backupconfiguration.stash.appscode.com "prod-1-app-sample-mysql-1" deleted
backupconfiguration.stash.appscode.com "prod-2-app-sample-mysql-2" deleted
backupconfiguration.stash.appscode.com "prod-3-app-sample-mysql-3" deleted
$ kubectl delete repository -n backup --all
repository.stash.appscode.com "prod-1-app-sample-mysql-1" deleted
repository.stash.appscode.com "prod-2-app-sample-mysql-2" deleted
repository.stash.appscode.com "prod-3-app-sample-mysql-3" deleted
$ kubectl delete secret -n backup gcs-secret
secret "gcs-secret" deleted
$ kubectl delete sa -n backup cross-namespace-target-reader
serviceaccount "mysql-sa" deleted
$ kubectl delete clusterrole cross-namespace-target-clusterrole
role.rbac.authorization.k8s.io "mysql-role" deleted
$ kubectl delete clusterrolebinding cross-namespace-target-clusterrolebinding
rolebinding.rbac.authorization.k8s.io "mysql-rolebinding" deleted
$ kubectl delete my -n prod-1 sample-mysql-1
mysql.kubedb.com "sample-mysql-1" deleted
$ kubectl delete my -n prod-2 sample-mysql-2
mysql.kubedb.com "sample-mysql-2" deleted
$ kubectl delete my -n prod-3 sample-mysql-3
mysql.kubedb.com "sample-mysql-3" deleted
```
