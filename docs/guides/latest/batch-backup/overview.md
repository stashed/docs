---
title: Batch Backup Overview | Stash
description: An overview on how batch backup in Stash.
menu:
  product_stash_{{ .version }}:
    identifier: batch-backup-overview
    name: How batch backup works?
    parent: batch-backup
    weight: 10
product_name: stash
menu_name: product_stash_{{ .version }}
section_menu_id: guides
---

# How Batch Backup Works in Stash

Sometimes, a single component may not meet the requirement for your application. For example, in order to deploy a WordPress, you will need a Deployment for the WordPress and another Deployment for database to store it's contents. Now, you may want to backup both of the deployment and database under a single configuration as they are parts of a single application.

A `BackupBatch` is a Kubernetes `CustomResourceDefinition`(CRD) which let you configure backup for multiple co-related workloads under a single configuration.

## How Backup Process Works

The following diagram shows how Stash takes backup of multiple co-related components in a single application. Open the image in a new tab to see the enlarged version.

<figure align="center">
  <img alt="Stash Backup Flow" src="/docs/images/guides/latest/batch-backup/batchbackup_overview.svg">
<figcaption align="center">Fig: Backup process of multiple targets(workload, PVC, database) in Stash</figcaption>
</figure>

The backup process consists of the following steps:

1. At first, a user creates a Secret. This secret holds the credentials to access the backend where the backed up data will be stored.

2. Then, she creates a `Repository` crd which represents the original repository in the backend.

3. Then, she creates a `BackupBatch` crd which specifies multiple targets(workload, volume,and database). It also specifies the `Repository` object that holds the backend information where the backed up data will be stored.

4. Stash operator watches for `BackupBatch` objects.

5. When it finds a `BackupBatch` object, it checks if there is any workload as a target. If there any, it injects a sidecar named `stash` into the workloads.

6. It also creates a `CronJob` to trigger backups periodically.

7. The`CronJob` triggers backup on each scheduled slot by creating a `BackupSession` crd.
  
8. The BackupSession controller (inside sidecar for sidecar model or inside the operator itself for job model) watches for `BackupSession` crd.

9. When it finds a `BackupSession` it starts the backup process immediately(for job model a job is created for taking backup) for the individual targets.
  
10. The individual targets complete their backup process independently and update their respective fields in `BackupSession` status.

## Next Steps

- See a step by step guide to backup application with multiple co-related components [here](/docs/guides/latest/batch-backup/batch-backup.md)