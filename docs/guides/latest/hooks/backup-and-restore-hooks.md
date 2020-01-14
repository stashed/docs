---
title: Backup & Restore Hooks | Stash
menu:
  docs_{{ .version }}:
    identifier: backup-and-restore-hooks
    name: Backup & Restore Hooks
    parent: hooks
    weight: 20
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: guides
---

# Backup & Restore Hooks

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [kind](https://kind.sigs.k8s.io/docs/user/quick-start/).
- Install Stash in your cluster following the steps [here](/docs/setup/install.md).
- Install MySQL addon for Stash following the steps [here](/docs/addons/mysql/setup/install.md).
- Install [KubeDB](https://kubedb.com) in your cluster following the steps [here](https://kubedb.com/docs/setup/install/). This step is optional. You can deploy your database using any method you want. We are using KubeDB because KubeDB simplifies many of the difficult or tedious management tasks of running a production grade databases on private and public clouds.
- If you are not familiar with how Stash backup and restore MySQL databases, please check the following guide [here](/docs/addons/mysql/overview.md).

You should be familiar with the following `Stash` concepts:

- [BackupBlueprint](/docs/concepts/crds/backupblueprint.md)
- [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md)
- [BackupSession](/docs/concepts/crds/backupsession.md)
- [Repository](/docs/concepts/crds/repository.md)
- [Function](/docs/concepts/crds/function.md)
- [Task](/docs/concepts/crds/task.md)

To keep everything isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$ kubectl create ns demo
namespace/demo created
```

## Prepare Database

**Deploy Database:**

**Verify AppBinding:**

**Insert Sample Data:**

## Prepare Backend

**Create Storage Secret:**

**Create Repository:**

## PreBackup Hook

## PostBackup Hook

## PreRestore Hook

## PostRestore Hook

## Cleanup
