---
title: Percona XtraDB Backup & Restore Overview | Stash
description: How Percona XtraDB Backup & Restore Works in Stash
menu:
  product_stash_{{ .version }}:
    identifier: stash-percona-xtradb-overview
    name: How does it works?
    parent: stash-percona-xtradb
    weight: 10
product_name: stash
menu_name: product_stash_{{ .version }}
section_menu_id: stash-addons
---

# How Stash Backup & Restore Percona XtraDB Database

Stash 0.9.0+ supports backup and restore operation of many databases. This guide will give you an overview of how Percona XtraDB database backup and restore process works in Stash.

## How Backup Works

### Backup Percona XtraDB Cluster

The following diagram shows how Stash takes from Percona XtraDB Cluster backup. Open the image in a new tab to see the enlarged version.

<figure align="center">
  <img alt="Percona XtraDB Cluster Backup Overview" src="/docs/images/addons/percona-xtradb/backup_overview.svg">
  <figcaption align="center">Fig: Percona XtraDB Cluster Backup Overview</figcaption>
</figure>

The backup process consists of the following steps:

1. At first, a user creates a secret with access credentials of the backend where the backed up data will be stored.

2. Then, she creates a `Repository` crd that specifies the backend information along with the secret that holds the credentials to access the backend.

3. Then, she creates a `BackupConfiguration` crd targeting the [AppBinding](/docs/concepts/crds/appbinding.md) crd of the desired database. The `BackupConfiguration` object also specifies the `Task` to use to backup the database.

4. Stash operator watches for `BackupConfiguration` crd.

5. Once Stash operator finds a `BackupConfiguration` crd, it creates a CronJob with the schedule specified in `BackupConfiguration` object to trigger backup periodically.

6. On the next scheduled slot, the CronJob triggers a backup by creating a `BackupSession` crd.

7. Stash operator also watches for `BackupSession` crd.

8. When it finds a `BackupSession` object, it resolves the respective `Task` and `Function` and prepares a Job definition to backup.

9. Then, it creates the Job to backup the targeted database.

10. The backup Job reads necessary information to connect with the database from the `AppBinding` crd. It also reads backend information and access credentials from `Repository` crd and Storage Secret respectively.

11. Then, the Job runs the backup procedure to take the backup of the targeted database and uploads the output to the backend. During backup procedure, the Job takes a full snapshot of the data directory (`/var/lib/mysql`). Stash pipes the output of the backup procedure to the uploading process. Hence, backup Job does not require a large volume to hold the entire backed up data.

12. Finally, when the backup is complete, the Job sends Prometheus metrics to the Pushgateway running inside Stash operator pod. It also updates the `BackupSession` and `Repository` status to reflect the backup procedure.

## How Restore Process Works

### Restore Percona XtraDB Cluster

The following diagram shows how Stash restores backed up data into a Percona XtraDB cluster. Open the image in a new tab to see the enlarged version.

<figure align="center">
  <img alt="Percona XtraDB Cluster Restore Overview" src="/docs/images/addons/percona-xtradb/restore_overview.svg">
  <figcaption align="center">Fig: Percona XtraDB Cluster Restore Process Overview</figcaption>
</figure>

The restore process consists of the following steps:

1. At first, a user creates a `RestoreSession` crd targeting the `AppBinding` of the desired database where the backed up data will be restored. It also specifies the `Repository` crd which holds the backend information and the `Task` to use to restore the target.

2. Stash operator watches for `RestoreSession` object.

3. Once it finds a `RestoreSession` object, it resolves the respective `Task` and `Function` and prepares a number (equal to the value of `.spec.target.replicas` of `RestoreSession` object) of Job definitions to restore. Each of these Job requires a PVC to store the data (full data directory `/var/lib/mysql`) from the backend.

4. Then, it creates these Jobs and corresponding PVCs to restore the target.

5. These Jobs read necessary information to connect with the database from respective `AppBinding` crd. They also read backend information and access credentials from `Repository` crd and Storage Secret respectively.

6. Then, each job downloads the backed up data from the backend and injects into the associated PVC.

7. Finally, when the restore process is complete, the Job sends Prometheus metrics to the Pushgateway and update the `RestoreSession` status to reflect restore completion.

## Next Steps

- Install Percona XtraDB addon for Stash following the guide from [here](/docs/addons/percona-xtradb/setup/install.md).
