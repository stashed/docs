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

Here, we are going to give you an overview of Stash documentation structure.

## [Concepts](/docs/concepts/)

Concept explains some significant aspect of Stash. This is where you can learn about what Stash does and how it does it.

- [Stash Overview](/docs/concepts/what-is-stash/overview.md) Provides an introduction to Stash and gives an overview of the features it provides.
- [Stash Architecture](/docs/concepts/what-is-stash/architecture.md) Provides an overview of Stash architecture and it's core components.
- [Stash API](/docs/concepts/crds/repository.md) Introduces Stash CRDs.

## [Setup](/docs/setup/)

Setup contains instruction for installing, uninstalling, and upgrading Stash.

- **Install Stash:** Provides installation instructions for Stash and its various components.
  - [Community Edition](/docs/setup/install/community.md): Provides installation instructions for Stash Community Edition.
  - [Enterprise Edition](/docs/setup/install/enterprise.md): Provides installation instructions for Stash Enterprise Edition.
  - [Stash kubectl Plugin](/docs/setup/install/kubectl_plugin.md): Provides installation instructions for Stash `kubectl` plugin.
  - [Troubleshooting](/docs/setup/install/troubleshoting.md): Provides troubleshooting guide for various installation problems.
- **Uninstall Stash:** Provides uninstallation instructions for Stash and its various components.
  - [Community Edition](/docs/setup/uninstall/community.md): Provides uninstallation instructions for Stash Community Edition.
  - [Enterprise Edition](/docs/setup/uninstall/enterprise.md): Provides uninstallation instructions for Stash Enterprise Edition.
  - [Stash kubectl Plugin](/docs/setup/uninstall/kubectl_plugin.md): Provides uninstallation instructions for Stash `kubectl` plugin.
- [Upgrade Stash](/docs/setup/upgrade/index.md): Provides instruction for updating Stash license and upgrading between various Stash versions.

## [Guides](/docs/guides/)

Guides show how to perform different operations with Stash.

- [Supported Backends](/docs/guides/backends/overview.md): Describes how to configure different storage for storing backed up data.
- [Workload Volume Backup](/docs/guides/workloads/overview.md): Shows how to use Stash to backup and restore volumes of a workload (i.e. `Deployment`, `StatefulSet`, `DaemonSet`, etc).
- [Stand-alone Volume Backup](/docs/guides/volumes/overview.md): Shows how to use Stash to backup and restore stand-alone volumes(i.e. `PersistentVolumeClaim`).
- [Batch Backup](/docs/guides/batch-backup/overview.md): Shows how to backup multiple co-related components using a single configuration known as `BackupBatch`.
- [Auto Backup](/docs/guides/auto-backup/overview.md): Shows how to configure automatic backup of any stateful workload in your cluster.
- [Volume Snapshot](/docs/guides/volumesnapshot/overview/index.md): Shows how Stash takes snapshot of `PersistentVolumeClaim`s and restore them from snapshot using Kubernetes `VolumeSnapshot` API.

- **Different Use Cases:**
Shows different uses cases of Stash like instant backup, pause backup, cross-namespace backup and restore etc.

  - [Instant Backup](/docs/guides/use-cases/instant-backup.md): Shows you how to take an instant backup in Stash.
  - [Exclude/Include Files](/docs/guides/use-cases/exclude-include-files/index.md): Shows how to exclude/include subset of files during backup/restore process.
  - [Pause Backup](/docs/guides/use-cases/pause-backup.md): Shows how to pause a backup temporarily.
  - [Clone Data Volumes](/docs/guides/use-cases/clone-pvc.md): Shows how to clone data volumes of a workload into a different namespace in a cluster using Stash.
  - [File Ownership](/docs/guides/use-cases/ownership.md): Explains when and how ownership of the restored files can be changed.
  - [Cross-Cluster Backup and Restore](/docs/guides/use-cases/cross-cluster-backup/index.md): Shows how to take backup and restore across clusters using Stash.
  - [Customize Backup and Restore](/docs/guides/use-cases/customize-backup-restore/index.md): Shows how to customize backup and restore processes in Stash according to your needs.

- **Managed Backup**
Shows how backup and restore can be managed in some specific scenerios.
  - [Dedicated Backup Namespace](/docs/guides/managed-backup/dedicated-backup-namespace/index.md): Shows you how to manage backup and restore from a dedicated namespace for targets of different namespaces using Stash.
  - [Dedicated Storage Namespace](/docs/guides/managed-backup/dedicated-storage-namespace/index.md): Shows you how to take backup and restore by keeping the storage resources (Repository and backend Secret) in a dedicated namespace using Stash.

- [Platforms](/docs/guides/platforms/eks-irsa/index.md): Shows how to use Stash to backup and restore volumes of a Kubernetes workload running in different platforms.
- [Monitoring](/docs/guides/monitoring/overview/index.md): Shows how Prometheus monitoring works with Stash, what metrics Stash exports, and how to enable monitoring.
- [Hooks](/docs/guides/hooks/overview/index.md): Shows how to execute different actions before/after the backup/restore process.
- [CLI](/docs/guides/cli/cli.md): Shows how to manage Stash objects quickly and easily using Stash `kubectl` plugin.
- [Troubleshooting](/docs/guides/troubleshooting/how-to-troubleshoot/index.md): Gives an overview of how you can gather the necessary information to identify the issue that causes the backup/restore failure.
- [Security](/docs/guides/security/rbac.md): Describes different built-in cluster security support by Stash.

We're always looking for help improving our documentation, so please don't hesitate to [file an issue](https://github.com/stashed/project/issues/new) if you see some problem.
