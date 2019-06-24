---
title: Default Backup Overview | Stash
description: An overview on how default backup works in Stash.
menu:
  product_stash_0.8.3:
    identifier: default-backup-overview
    name: What is Default Backup?
    parent: default-backup
    weight: 10
product_name: stash
menu_name: product_stash_0.8.3
section_menu_id: guides
---

# Default Backup in Stash

This guide will show you what is default backup and how default backup works in Stash.

## What is Default Backup?

Stash uses 1-1 mapping among `Repository`, `BackupConfiguration` and the target. So, whenever you want to backup a target(workload/PVC/database), you have to create a `Repository` and `BackupConfiguration` object. This could become tiresome when you are trying to backup similar types of target and the `Repository` and `BackupConfiguration` has only a slight difference. To mitigate this problem, Stash provides a way to specify a template for these two objects via `BackupConfigurationTemplate` crd. In Stash parlance, we call this process as **default backup**.

You have to create only one `BackupConfigurationTemplate` for all similar types of target. For example, you need only one `BackupConfigurationTemplate` for Deployment, DaemonSet, StatefulSet etc. Similarly, you have to create only one `BackupconfigurationTemplate` for all PostgreSQL databases. Then, you just need to add some annotations in the target. Stash will automatically create respective `Repository`, `BackupConfiguration` object using the template.

## How Default Backup Works?

The following diagram shows how default backup works in Stash. Open the image in new tab to see enlarged image.

<figure align="center">
  <img alt="Default Backup Overview" src="/docs/images/guides/latest/default-backup/default_backup.svg">
  <figcaption align="center">Fig: Default Backup Overview</figcaption>
</figure>

The default backup process consists of the following steps:

1. A user creates a storage secret with necessary credentials of the backend where the backed up data will be stored.
2. Then, he creates a `BackupConfigurationTemplate` crd that specifies a template for `Repository` and `BackupConfiguration` object.
3. Then, he creates a workload with some specific annotations for default backup.
4. Stash operator watches for workloads. When it finds a workload with default backup specific annotations, it finds out the respective `BackupConfigurationTemplate`.
5. Then, it resolves the template by replacing variable fields of the template with respective information from the workload.
6. Then, it creates a `Repository` and a `BackupConfiguration` object for the workload according to the resolved template.
7. Finally, Stash start rest of the standard backup process as discussed in [here](/docs/guides/latest/workload/overview.md).

## Next Step

- Learn how to configure default backup for workloads from [here](/docs/guides/latest/default-backup/workload.md).
- Learn how to configure default backup for PVCs from [here](/docs/guides/latest/default-backup/pvc.md).
- Learn how to configure default backup for databases from [here](/docs/guides/latest/default-backup/database.md).
