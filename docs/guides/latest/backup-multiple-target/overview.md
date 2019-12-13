---
title: Multiple Target Backup Overview | Stash
description: An overview on how backup multiple target data in Stash.
menu:
  product_stash_{{ .version }}:
    identifier: backup-overview
    name: How multiple target backup works?
    parent: backup-multiple-target
    weight: 10
product_name: stash
menu_name: product_stash_{{ .version }}
section_menu_id: guides
---

# Backup Multiple Target using Stash

If you want to take backup multiple targets (workloads, databases, PVCs, etc.) simultaneously, You can do it by using `BackupBatch` crd in stash. Stash gives you multiple target backup support simultaneously. This guide will show you how Stash backs up multiple targets data.

## Before You Begin

- You should be familiar with the following `Stash` concepts:
  - [BackupBatch](/docs/concepts/crds/backupbatch.md)
  - [BackupSession](/docs/concepts/crds/backupsession.md)
  - [Repository](/docs/concepts/crds/repository.md)

## How Backup Process Works

The following diagram shows how Stash takes backup of multiple targets(workload, PVC, database). Open the image in a new tab to see the enlarged version.

<figure align="center">
  <img alt="Stash Backup Flow" src="/docs/images/guides/latest/backup-multiple-target/backupbatch_overview.svg">
<figcaption align="center">Fig: Backup process of multiple targets(workload, PVC, database) in Stash</figcaption>
</figure>

The backup process consists of the following steps:

- At first, a user creates a Secret. This secret holds the credentials to access the backend where the backed up data will be stored.(1)

- Then, she creates a `Repository` crd which represents the original repository in the backend.(2)

- Then, she creates a `BackupBatch` crd which specifies multiple target (workload, volume and database). It also specifies the `Repository` object that holds the backend information where the backed up data will be stored.(3)

- Stash operator watches for `BackupBatch` objects.(4)

- When it finds a `BackupBatch` object, it finds the targeted workloads and injects a sidecar named `stash` into the workloads.(5)

- It also creates a `CronJob` to trigger backups periodically.(6)

- The`CronJob` triggers backup on each scheduled slot by creating a `BackupSession` crd.(7)
  
- The operator(8a) and The `stash` sidecar(8b) inside the workload watches for `BackupSession` crd.

When They finds a `BackupSession` crd, two models(sidecar and job) initiates backup process independently and simultaneously.

Sidecar Model :

- The sidecar takes backup of the targeted file paths. Once the backup process is completed, the `sidecar` sends Prometheus metrics to the Pushgateway running inside the `stash-operator` pod. It also updates respective `BackupSession` and `Repository` status to reflect the backup process.(10)

Job Model :

- The operator resolves the respective `Task` and `Function` and prepares a backup Job definition. Then, it mounts the targeted volume(database and PVC) into the Job and creates it.(9a,9b)

- The Job takes backup of the targeted volume. Finally, when backup is completed, the Job sends Prometheus metrics to the Pushgateway running inside Stash operator pod. It also updates the `BackupSession` and `Repository` status to reflect the backup procedure.(10a,10b)

## Next Steps

1. See a step by step guide to backup volumes of Workloads [here](/docs/guides/latest/backup-multiple-target/workloads.md).
2. See a step by step guide to backup volumes of workloads and PVC [here](/docs/guides/latest/backup-multiple-target/volumes.md).
