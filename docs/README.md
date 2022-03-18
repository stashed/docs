---
title: Welcome | Stash
description: Welcome to Stash
menu:
  docs_{{ .version }}:
    identifier: readme-stash
    name: Readme
    parent: welcome
    weight: -1
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: welcome
url: /docs/{{ .version }}/welcome/
aliases:
  - /docs/{{ .version }}/
  - /docs/{{ .version }}/README/
---

# Stash

[Stash](https://stash.run) by AppsCode is a cloud native data backup and recovery solution for Kubernetes workloads. If you are running production workloads in Kubernetes, you might want to take backup of your disks, databases etc. Traditional tools are too complex to set up and maintain in a dynamic compute environment like Kubernetes. Stash is a Kubernetes operator that uses [restic](https://github.com/restic/restic) or Kubernetes CSI Driver VolumeSnapshotter functionality to address these issues. Using Stash, you can backup Kubernetes volumes mounted in workloads, stand-alone volumes and databases. Users may even extend Stash via [addons](https://stash.run/docs/{{< param "info.version" >}}/guides/addons/overview/) for any custom workload.

From here you can learn all about Stash's architecture and how to deploy and use Stash.

- [Concepts](/docs/concepts/) Explains some significant aspect of Stash. This is where you can learn about what Stash does and how it does it.
  - [What is Stash?](/docs/concepts/what-is-stash/overview.md) Provides an introduction to Stash and gives an overview of the features it provides.
    - [Architecture](/docs/concepts/what-is-stash/architecture.md) provides a visual representation of Stash architecture. It also provides a brief overview of the components it uses.
  - **Declarative API**
    - [Repository](/docs/concepts/crds/repository.md) introduces the concept of `Repository` crd that holds backend information in a Kubernetes native way.
    - [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md) introduces the concept of `BackupConfiguration` crd that is used to configure backup for a target resource in a Kubernetes native way.
    - [BackupBatch](/docs/concepts/crds/backupbatch.md) introduces the concept of `BackupBatch` crd that is used to setup backup of multiple co-related targets under single configuration.
    - [BackupSession](/docs/concepts/crds/backupsession.md) introduces the concept of `BackupSession` crd that represents a backup run of a target resource for the respective `BackupConfiguration` or `BackupBatch` object.
    - [RestoreSession](/docs/concepts/crds/restoresession.md) introduces the concept of `RestoreSession` crd that represents a restore run of a target resource.
    - [RestoreBatch](/docs/concepts/crds/restorebatch.md) introduces the concept of `RestoreBatch` crd that allows restore of multiple targets that were backed up using `BackupBatch` under single configuration.
    - [Function](/docs/concepts/crds/function.md) introduces the concept of `Function` crd that represents a step of a backup or restore process.
    - [Task](/docs/concepts/crds/task.md) introduces the concept of `Task` crd which specifies an ordered collection of multiple `Function`s and their parameters that make up a complete backup or restore process.
    - [BackupBlueprint](/docs/concepts/crds/backupblueprint.md) introduces the concept of `BackupBlueprint` crd that specifies a blueprint for `Repository` and `BackupConfiguration` object which provides an option to share backup configuration across similar targets.
    - [AppBinding](/docs/concepts/crds/appbinding.md) introduces the concept of `AppBinding` crd which holds the information that are necessary to connect with an application like database.
    - [Snapshot](/docs/concepts/crds/snapshot.md) introduces the concept of `Snapshot` object that represents backed up snapshots in a Kubernetes native way.

- [Setup](/docs/setup/) Contains instructions for installing
  the Stash in various cloud providers.
  - **Install Stash:** Installation instructions for Stash and its various components.
    - [Community Edition](/docs/setup/install/community.md): Installation instructions for Stash Community Edition.
    - [Enterprise Edition](/docs/setup/install/enterprise.md): Installation instructions for Stash Enterprise Edition.
    - [Stash kubectl Plugin](/docs/setup/install/kubectl_plugin.md): Installation instructions for Stash `kubectl` plugin.
    - [Troubleshooting](/docs/setup/install/troubleshoting.md): Troubleshooting guide for various installation problems.
  - **Uninstall Stash:** Uninstallation instructions for Stash and its various components.
    - [Community Edition](/docs/setup/uninstall/community.md): Uninstallation instructions for Stash Community Edition.
    - [Enterprise Edition](/docs/setup/uninstall/enterprise.md): Uninstallation instructions for Stash Enterprise Edition.
    - [Stash kubectl Plugin](/docs/setup/uninstall/kubectl_plugin.md): Uninstallation instructions for Stash `kubectl` plugin.
    - [Upgrading Stash](/docs/setup/upgrade/index.md): Instruction for updating Stash license and upgrading between various Stash versions.

- [Guides](/docs/guides/) Shows you how to perform tasks with Stash.
  - [Supported Backends](/docs/guides/backends/overview.md) Describes how to configure different backends for Stash.
    - [Kubernetes Volume](/docs/guides/backends/local.md) Describes how to configure local backend for Stash.
    - [AWS S3:](/docs/guides/backends/s3.md) Describes how to configure AWS S3 or S3 compatible storage services like Minio servers as a backend for Stash.
    - [Microsoft Azure Storage](/docs/guides/backends/azure.md) Describes how to configure  Azure Blob Storage as a backend for Stash.
    - [Google Cloud Storage](/docs/guides/backends/gcs.md) Describes how to configure Google Cloud Storage(GCS) as a backend for Stash.
    - [OpenStack Swift](/docs/guides/backends/swift.md) Describes how to configure OpenStack Swift as a backend for Stash.
    - [B2 Cloud Storage](/docs/guides/backends/b2.md) Describes how to configure B2 Cloud Storage as a backend for Stash.
    - [REST Backend](/docs/guides/backends/rest.md) Describes how to configure REST Backend as a backend for Stash.
  - [Workload Volumes](/docs/guides/workloads/overview.md) Shows how to use Stash to backup and restore volumes of a workload (`Deployment`, `StatefulSet`, `DaemonSet`).
    - [Backup and Restore Deployment](/docs/guides/workloads/deployment.md) Shows how to use Stash to backup and restore volumes of a `Deployment`.
    - [Backup and Restore Statefulset](/docs/guides/workloads/statefulset.md) Shows how to use Stash to backup and restore volumes of a `Statefulset`.
    - [Backup and Restore Daemonset](/docs/guides/workloads/daemonset.md) Shows how to use Stash to backup and restore volumes of a `Daemonset`.
  - [Stand-alone Volumes](/docs/guides/volumes/overview.md) Shows how to use Stash to backup and restore stand-alone volumes.
    - [Backup and Restore Stand-alone PVC using Stash](/docs/guides/volumes/pvc.md) Shows how to backup a stand-alone `PersistentVolumeClaim` (PVC) using Stash.
  - [Batch Backup](/docs/guides/batch-backup/overview.md) Shows how to backup multiple co-related components using a single configuration known as `BackupBatch`.
  - [Auto Backup](/docs/guides/auto-backup/overview.md) Shows how to configure automatic backup of any stateful workload in your cluster.
    - [Auto Backup for Workloads](/docs/guides/auto-backup/workload.md) Shows how to configure automatic backup for Kubernetes workloads.
    - [Auto Backup for Workloads](/docs/guides/auto-backup/pvc.md) Shows how to configure automatic backup for `PersistentVolumeClaim`s.
    - [Auto Backup for Workloads](/docs/guides/auto-backup/database.md) Shows how to configure automatic backup for databases.
  - [Volume Snapshot](/docs/guides/volumesnapshot/overview.md) shows how Stash takes snapshot of `PersistentVolumeClaim`s and restore them from snapshot using Kubernetes `VolumeSnapshot` API.
    - [Snapshot Deployment Volumes](/docs/guides/volumesnapshot/deployment.md) Shows how to use Stash to snapshot the volumes of a `Deployment` and restore them from snapshot using Kubernetes `VolumeSnapshot` API.
    - [Snapshot Statefulset Volumes](/docs/guides/volumesnapshot/statefulset.md) Shows how to use Stash to snapshot the volumes of a `Statefulset` and restore them from snapshot using Kubernetes `VolumeSnapshot` API.
    - [Snapshot Standalone PVC](/docs/guides/volumesnapshot/pvc.md) Shows how to use Stash to snapshot standalone `PersistentVolumeClaim`s and restore them from snapshot using Kubernetes `VolumeSnapshot` API.
  - **Advance Use Cases** Shows different advance uses cases of Stash like instant backup, pause backup, cross-namespace backup and restore etc.
    - [Instant Backup](/docs/guides/advanced-use-case/instant-backup.md) Shows you how to take an instant backup in Stash.
    - [Exclude/Include Files](/docs/guides/advanced-use-case/exclude-include-files/index.md) Shows how to exclude/include subset of files during backup/restore process.
    - [Pause Backup](/docs/guides/advanced-use-case/pause-backup.md) Shows how to pause a backup.
    - [Clone Data Volumes](/docs/guides/advanced-use-case/clone-pvc.md) Shows how to clone data volumes of a workload into a different namespace in a cluster using Stash.
    - [File Ownership](/docs/guides/advanced-use-case/ownership.md) explains when and how ownership of the restored files can be changed.
    - [Cross-Namespace Backup and Restore](/docs/guides/advanced-use-case/cross-namespace-backup/index.md) Shows how to take backup and restore across namespaces using Stash.
    - [Cross-Cluster Backup and Restore](/docs/guides/advanced-use-case/cross-cluster-backup/index.md) Shows how to take backup and restore across clusters using Stash.

We're always looking for help improving our documentation, so please don't hesitate to [file an issue](https://github.com/stashed/project/issues/new) if you see some problem.
