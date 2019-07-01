---
title: Backup and Restore Workload Data Overview | Stash
description: An overview on how Backup and Restore of workload data works in Stash.
menu:
  product_stash_0.8.3:
    identifier: workload-overview
    name: How Backup and Restore works?
    parent: workload
    weight: 10
product_name: stash
menu_name: product_stash_0.8.3
section_menu_id: guides
---

# Backup and Restore Workload's data using Stash

This guide will show you how Stash backup and restore workloads (Deployment, StatefulSet, DaemonSet etc.) data.

## Before You Begin

- You should be familiar with the following `Stash` concepts:
  - [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md/)
  - [BackupSession](/docs/concepts/crds/backupsession.md/)
  - [RestoreSession](/docs/concepts/crds/restoresession.md/)
  - [Repository](/docs/concepts/crds/repository.md/)

## How Backup Works?

The following diagram shows how Stash takes backup workload's data. Open the image in a new tab to see the enlarged image.

<figure align="center">
  <img alt="Stash Backup Flow" src="/docs/images/latest/workloads/backup_overview.svg">
<figcaption align="center">Fig: Backup Process of Workload's data in Stash</figcaption>
</figure>

The backup process consists of the following steps:

1. At first, a user creates a Secret. This secret holds the credentials to access the backend where the backed up data will be stored.

2. Then, she creates a `Repository` crd which represents the original repository in the backend.
  
3. Then, she creates a `BackupConfiguration` crd which specifies the targeted workload and desired directories to backup. It also specifies the `Repository` object that holds the backend information where the backed up data will be stored.

4. Stash operator watches for `BackupConfiguration` object.

5. Once it found a `BackupConfiguration` object, it find the targeted workload and injects a sidecar named `stash` to backup.

6. It also creates a `CronJob` to trigger backup periodically.  

7. The`CronJob` triggers backup on each schedule by creating a `BackupSession` crd.

8. The `stash` sidecar inside the workload watches for `BackupSession` crd.

9. When it found a `BackupSession` crd, it instantly takes a backup of the targeted directories.

10. Once the backup process is completed, the `sidecar` send Prometheus metrics to the Pushgateway running inside the `stash-operator` pod. It also update respective `BackupSession` and `Repository` status to reflect the backup.

## How Restore Works?

The following diagram shows how Stash restores backed up data inside a workload. Open the image in a new tab to see the enlarged image.

<figure align="center">
  <img alt="Stash Backup Flow" src="/docs/images/latest/workloads/restore_overview.svg">
<figcaption align="center">Fig: Restore Process of Workload's data in Stash</figcaption>
</figure>

The restore process consists of the following steps:

1. At first, the user creates a workload where the data will be restored.

2. Then, she creates a `RestoreSession` crd that specifies the targeted workload where the backed up data will be restored. It also specifies the respective `Repository` that holds the respective backend information.

3. Stash operator watches for `RestoreSession` crd.

4. Once it found a `RestoreSession` crd, it injects an init-container named `stash-init` to the workload and restart it.

5. The init-container restores the desired data from the backend on start-up.

6. Finally, when the restore process is completed it send Prometheus metrics to the `pushgateway` running inside the stash operator. It also update the `RestoreSession` status to reflect the restore process.

>**Note:** If your workload restart with the `stash-init` init-container for any reason, the init-container will skip running restore process if there is no pending `RestoreSession` for this workload.

# Nest Steps

1. See a step by step guide to backup/restore a Deployment's volumes from [here](docs/guides/latest/workloads/deployment.md).

2. See a step by step guide to backup/restore a StatefulSet's volumes from [here](docs/guides/latest/workloads/statefulset.md).

3. See a step by step guide to backup/restore a Daemonset's volumes from [here](docs/guides/latest/workloads/daemonset.md).