# Backup and Restore volumes using Stash

This guide will show you how Stash backup and restore a kubernetes volumes using Stash.

## Before You Begin

- At first, you need to have a Kubernetes cluster, and the `kubectl` command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using Minikube.

- Install `Stash` in your cluster following the steps [here](https://appscode.com/products/stash/0.8.3/setup/install/).

- You should be familiar with the following `Stash` concepts:
  - [BackupConfiguration](/docs/concepts/crds/backupconfiguration.md/)
  - [BackupSession](/docs/concepts/crds/backupsession.md/)
  - [RestoreSession](/docs/concepts/crds/restoresession.md/)
  - [Repository](/docs/concepts/crds/repository.md/)

## How Backup Works?

The following diagram shows how Stash takes backup of a Kubernetes volume. Open the image in a new tab to see the enlarged image.

<figure align="center">
  <img alt="Stash Backup Flow" src="/docs/images/v1beta1/backends/workloads/backup_overview.svg">
<figcaption align="center">Fig: Backup Process in Stash</figcaption>
</figure>

The backup process consists of the following steps:

1. At first, a user creates a Secret. This secret holds the credentials to access the backend where backed up data will be stored.

2. The user creates a `Repository` crd which represents the original repository in the backend.
  
3. Then, the user creates a `BackupConfiguration` crd which specifies the targeted workload for backup and the `Repository` object that holds backend infomed `stash` and mounts the target volumes into it.

4. Once it found a `BackupConfiguration` crd, it injects a sidecar container named `stash` and mounts the target volumes into it.

5. The Stash operator creates a `CronJob` to take a perioddic backup of the target volumes.  

6. The`CronJob` triggers backup on each schedule by creating a `BackupSession` crd.

7. The sidecar watches for `BackupSession` crd.

8. Once it found a `BackupSession` crd, it takes periodic backup of the volume to specified backend.

9.  The sidecar send metrics to Prometheus pushgateway in the stash operator. Once the backup is completed, stash operator updates the `status.phase` of the `BackupSession` crd.

## How Restore Works?

The following diagram shows how Stash restores backed up data from a backend. Open the image in a new tab to see the enlarged image.

<figure align="center">
  <img alt="Stash Backup Flow" src="/docs/images/v1beta1/backends/workloads/restore_overview.svg">
<figcaption align="center">Fig: Restore Process in Stash</figcaption>
</figure>

The restore process consists of the following steps:

1. At first, a user creates a workload to restore data on it's volume.

2. Then the user creates a `RestoreSession` crd that specifies the target `Repository` from where she want to restore. It also specifies targeted workload where the recovered data will be restored.

3. Stash operator watches for `RestoreSession` crd.

4. Once it found a `RestoreSession` crd, it injects a `init-container` named `stash`. The `init-container` reads the backend information from `Repository` crd and the backend credentials from the storage `Secret`.

5. Then, the `init-container` restores data from the backend and stores it in the target volume.

6. The `init-container` send metrics to Prometheus pushgateway in the stash operator. Once the restore is completed, stash operator updates the `status.phase` of the `RestoreSession` crd.

# Nest Steps

1. See a step by step guide to backup/restore a Deployment's volumes from [here](docs/guides/latest/workloads/deployment.md).

2. See a step by step guide to backup/restore a StatefulSet's volumes from [here](docs/guides/latest/workloads/statefulset.md).

3. See a step by step guide to backup/restore a Daemonset's volumes from [here](docs/guides/latest/workloads/daemonset.md).