---
title: Etcd Backup & Restore Overview | Stash
description: How Etcd Backup & Restore Works in Stash
menu:
  docs_{{ .version }}:
    identifier: stash-etcd-overview
    name: How does it works?
    parent: stash-etcd
    weight: 10
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: stash-addons
---

{{< notice type="warning" message="This is an Enterprise-only feature. Please install [Stash Enterprise Edition](/docs/setup/install/enterprise.md) to try this feature." >}}

# How Stash Backups & Restores Etcd Database

Stash `{{< param "info.version" >}}` supports backup and restore operation of many databases. This guide will give you an overview of how Etcd database backup and restore process works in Stash.

## Backup

Stash supports taking backup of Etcd database using [etcdctl](https://github.com/etcd-io/etcd/tree/main/etcdctl). It is the most flexible way to perform backup and restore of Etcd database.

### How Backup Works

The following diagram shows how Stash takes backup of a Etcd database. Open the image in a new tab to see the enlarged version.

<figure align="center">
  <img alt="Etcd Backup Overview" src="/docs/addons/etcd/overview/images/etcd-backup.svg">
  <figcaption align="center">Fig: Etcd Backup Overview</figcaption>
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

10. The backup Job reads the necessary information to connect with the database from the `AppBinding` crd. It also reads backend information and access credentials from `Repository` crd and Storage Secret respectively.

11. Then, the Job takes snapshot of the targeted database using `etcdctl` and uploads the snapshot to the backend. Stash stores the snapshot temporarily in a directory before uploading that into the backend. You can limit the temporary directory size using `spec.TempDir` field of `BackupConfiguration` crd.

12. Finally, when the backup is complete, the Job sends Prometheus metrics to the Pushgateway running inside Stash operator pod. It also updates the `BackupSession` and `Repository` status to reflect the backup completion.

### How Restore from Backup Works

The following diagram shows how Stash restores an Etcd database from a backup. Open the image in a new tab to see the enlarged version.

<figure align="center">
  <img alt="Etcd Database Restore Overview" src="/docs/addons/etcd/overview/images/etcd-restore.svg">
  <figcaption align="center">Fig: Etcd Restore Process Overview</figcaption>
</figure>

The restore process consists of the following steps:

1. At first, a user creates a `RestoreSession` crd targeting the `AppBinding` of the desired database where the backed up data will be restored. It also specifies the `Repository` crd which holds the backend information and the `Task` to use to restore the target.

2. Stash operator watches for `RestoreSession` object.

3. Once it finds a `RestoreSession` object, it resolves the respective `Task` and `Function` and prepares a Job definition to restore.

4. Then, it creates the Job to restore the target.

5. The Job reads necessary information to connect with the database from respective `AppBinding` crd. It also reads backend information and access credentials from `Repository` crd and Storage Secret respectively.

6. Then, the job downloads the backed up snapshot from the backend and restore that snapshot into the `Etcd` database. Stash stores the downloaded files temporarily before inserting into the targeted database.

7. Finally, when the restore process is complete, the Job sends Prometheus metrics to the Pushgateway and update the `RestoreSession` status to reflect restore completion.

## Next Steps

- Backup your Etcd database using Stash following the guide from [here](/docs/addons/etcd/basic-auth/index.md).
