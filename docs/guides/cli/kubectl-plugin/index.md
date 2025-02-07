---
title: kubectl Plugin | Stash
menu:
  docs_{{ .version }}:
    identifier: stash-cli
    name: Stash kubectl Plugin
    parent: cli
    weight: 10
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: guides
---

> New to Stash? Please start [here](/docs/concepts/README.md).

# Stash kubectl Plugin

Stash gives you kubectl plugin support named `kubectl stash` cli. `kubectl stash` cli can be used to manage Stash objects quickly and easily. It performs various operations like creating Stash objects, coping Stash objects, cloning PVC, unlock Repository, triggering an instant backup, etc. To install Stash kubectl plugin on your workstation, follow the steps [here](/docs/setup/README.md).

## Available Command

Available command for `kubectl stash` cli are:

| Main Command                                       | Uses                                                                        |
|----------------------------------------------------|-----------------------------------------------------------------------------|
| [create repository](#create-repository)            | Create a new `Repository`.                                                  |
| [create backupconfig](#create-backupconfiguration) | Create a new `BackupConfiguration`.                                         |
| [create restoresession](#create-restoresession)    | Create a new `RestoreSession`.                                              |
| [cp secret](#copy-secret)                          | Copy `Secret` from source namespace to destination namespace.               |
| [cp repository](#copy-repository)                  | Copy `Repository` from source namespace to destination namespace.           |
| [copy backupconfig](#copy-backupconfiguration)     | Copy `BackupConfiguration` from source namespace to destination namespace.  |
| [copy volumesnapshot](#copy-volumesnapshot)        | Copy `VolumeSnapshot` from source namespace to destination namespace.       |
| [clone pvc](#clone-pvc)                            | Clone a PVC from source namespace to destination namespace.                 |
| [download](#download-snapshots)                    | Download backup snapshots from backend into your local repository.          |
| [trigger](#trigger-an-instant-backup)              | Take an instant backup.                                                     |
| [pause backup](#pause-backup)                      | Pause Stash backup.                                                         |
| [resume backup](#resume-backup)                    | Resume Stash backup.                                                        |
| [debug backup](#debug-backup)                      | Debug Stash backup issues.                                                  |
| [debug restore](#debug-restore)                    | Debug Stash restore issues.                                                 |
| [debug operator](#debug-operator)                  | Debug Stash operator issues.                                                |
| [key list](#key-list)                              | List the keys (passwords) of a restic repository.                           |
| [key add](#key-add)                                | Add a new key (password) to a restic repository.                            |
| [key update](#key-update)                          | Update current key (password) of a restic repository.                       |
| [key remove](#key-remove)                          | Remove a key (password) of a restic repository.                             |
| [gen rules](#generate-rules)                       | Generate restore rules from nearest snapshots at a specific time.           |
| [check](#check-repository)                         | Test the restic repository for errors and reports any errors it finds.      |
| [rebuild-index](#rebuild-index)                    | Create a new index based on the pack files in the restic repository         |
| [migrate](#migrate-repository)                     | Check the repository integrity and then upgrade the repository version to 2 |
| [prune](#prune)                                    | Remove unneeded data from the repository                                    |

## Create Command

`kubectl stash create` command is used to create stash objects. It creates various objects like `Repository`, `BackupConfiguration` and `RestoreSession` etc.

### Create Repository

To create a `Repository`, you need to provide a `Repository` name and backend information and credential. You will provide the information and credential by using flags. The available flags are:

| Flag                | Description                                                                   |
|---------------------|-------------------------------------------------------------------------------|
| `--namespace`       | Indicates the namespace where the Repository will be created                  |
| `--secret`          | Specify the name of the storage secret that will be used to create Repository |
| `--bucket`          | Specify the name of the cloud bucket/container.                               |
| `--prefix`          | Prefix denotes the directory inside the backend.                              |
| `--provider`        | Specify backend provider (i.e. gcs, s3, azure etc)                            |
| `--endpoint`        | Endpoint for s3/s3 compatible backend                                         |
| `--max-connections` | Specify maximum concurrent connections for GCS, Azure and B2 backend.         |

**Format:**

```bash
kubectl stash create <repository-name> [flags]
```

**Example:**

```bash
$ kubectl stash create repository gcs-repo --namespace=demo --secret=gcs-secret --bucket=appscode-qa --prefix=/source/data --provider=gcs
```

### Create BackupConfiguration

To create a `BackupConfiguration`, you need to provide `BackupConfiguration` name, `Repository` name, Target, and RetentionPolicy, etc. You will provide the `Repository` name, Target, RetentionPolicy by using flags. The available flags are:

| Flag                    | Description                                                                                    |
|-------------------------|------------------------------------------------------------------------------------------------|
| `--namespace`           | Indicates the namespace where the `BackupConfiguration` will be created                        |
| `--target-apiversion`   | Specify API-Version of the target resource.                                                    |
| `--target-kind`         | Specify kind of the target resource.                                                           |
| `--target-name`         | Specify name of the target resource.                                                           |
| `--repo-name`           | Specify name of the `Repository` that will be created.                                         |
| `--repo-namespace`      | Specify namespace of the `Repository` that will be created.                                    |
| `--schedule`            | Specify schedule of the backup.                                                                |
| `--driver`              | `Driver` indicates the mechanism used to backup (i.e. VolumeSnapshotter, Restic)               |
| `--task`                | Specify name of a `Task`                                                                       |
| `--volumesnpashotclass` | Specify name of the `VolumeSnapshotClass`.                                                     |
| `--replica`             | Replica specifies the number of replicas whose data should be backed up.                       |
| `--paths`               | A list of path that will be backed up                                                          |
| `--volume-mounts`       | Specify a list of volumes and their mountPaths                                                 |
| `--keep-last`           | Never delete the n last (most recent) snapshots.                                               |
| `--keep-hourly`         | For the last n hours in which a snapshot was made, keep only the last snapshot for each hour.  |
| `--keep-daily`          | For the last n days which have one or more snapshots, only keep the last one for that day.     |
| `--keep-weekly`         | For the last n weeks which have one or more snapshots, only keep the last one for that week.   |
| `--keep-monthly`        | For the last n months which have one or more snapshots, only keep the last one for that month. |
| `--keep-yearly`         | For the last n years which have one or more snapshots, only keep the last one for that year.   |
| `--prune`               | If set `true`, Stash will cleanup unreferenced data from the backend.                          |
| `--dry-run`             | Stash will not remove anything but print which snapshots would be removed.                     |

> Note: You must provide `<volumenName>:<mountPath>:<subPath>` in the `--volume-mounts` flag to specify the volumes and their mountPath and subPath. The `:<subPath>` part is optional.

**Format:**

```bash
kubectl stash create <backupconfig-name> [flags]
```

**Example:**

```bash
$ kubectl stash create backupconfig ss-backup --namespace=demo --repo-name=gcs-repo --schedule="*/4 * * * *" --target-apiversion=apps/v1 --target-kind=StatefulSet --target-name=stash-demo --paths=/source/data --volume-mounts=source-data:/source/data --keep-last=5 --prune=true
```

### Create RestoreSession

To create a `RestoreSession`, you need to provide a `Repository` name, Target or `VolumeClaimTemplate`, etc. You will provide the `Repository` name, Target or `VolumeClaimTemplate` by using flags. The available flags are:

| Flag                   | Description                                                                    |
|------------------------|--------------------------------------------------------------------------------|
| `--namespace`          | Indicates the namespace where the `RestoreSession` will be created             |
| `--target-apiversion`  | API-Version of the target resource.                                            |
| `--target-kind`        | Specify kind of the target resource.                                           |
| `--target-name`        | Specify name of the target resource.                                           |
| `--repo-name`          | specify name of the `Repository`.                                              |
| `--repo-namespace`     | specify namespace of the `Repository`                                          |
| `--driver`             | Driver indicates the mechanism used to backup (i.e. VolumeSnapshotter, Restic) |
| `--task`               | Name of the Task                                                               |
| `--replica`            | Replica specifies the number of replicas whose data should be backed up.       |
| `--paths`              | A list of path that will be backed up                                          |
| `--volume-mounts`      | A list of volumes and their mountPaths                                         |
| `--snapshots`          | Specify the name of the Snapshot(single)                                       |
| `--host`               | Specify the name of the Source host                                            |
| `--claim.name`         | Specify the name of the `VolumeClaimTemplate`                                  |
| `--claim.access-modes` | Access mode of the VolumeClaimTemplates                                        |
| `--claim.storageclass` | Specify the name of the Storage secret for VolumeClaimTemplate                 |
| `--claim.size`         | Total requested size of the VolumeClaimTemplate                                |
| `--claim.datasource`   | DataSource of the VolumeClaimTemplate                                          |

> Note: You must provide `<volumenName>:<mountPath>:<subPath>` in the `--volume-mounts` flag to specify the volumes and their mountPath and subPath. The `:<subPath>` part is optional.

**Format:**

```bash
kubectl stash create restoresession <restoresession-name> [flags]
```

**Example:**

```bash
$ kubectl stash create restoresession ss-restore --namespace=demo --repo-name=gcs-repo --target-apiversion=apps/v1 --target-kind=StatefulSet --target-name=stash-recovered --paths=/source/data --volume-mounts=source-data:/source/data
```

## Copy Command

`kubectl stash cp` command is used to copy stash objects from one namespace to another namespace. It copies various objects like `Secret`, `Repository`, `BackupConfiguration` and `VolumeSnapshot` etc.

### Copy Secret

To copy a Secret, you need to provide Secret name and destination namespace. You will provide the destination namespace by using flag. The available flags are:

| Flag             | Description                                                            |
|------------------|------------------------------------------------------------------------|
| `--namespace`    | Indicates the namespace of the respective `Secret`.                    |
| `--to-namespace` | Indicates the destination namespace where the `Secret` will be copied. |

**Format:**

```bash
kubectl stash cp secret <secret-name> [flags]
```

**Example:**

```bash
$ kubectl stash cp secret my-secret --namespace=demo --to-namespace=demo1
```

### Copy Repository

To copy a Repository, you need to provide a Repository name and destination namespace. When we run the command the coping process consists of the following steps:
  
- At first, the cli copies Secret from source namespace to destination namespace.
- Then it copies Repository from source namespace to destination namespace.

You will provide the destination namespace by using flag. The available flags are:

| Flag             | Description                                                                |
|------------------|----------------------------------------------------------------------------|
| `--namespace`    | Indicates the namespace of the respective `Repository`.                    |
| `--to-namespace` | Indicates the destination namespace where the `Repository` will be copied. |

**Format:**

```bash
kubectl stash cp repository <repository-name> [flags]
```

**Example:**

```bash
$ kubectl stash cp repository my-repo --namespce=demo --to-namespace=demo1
```

### Copy BackupConfiguration

To copy a BackupConfiguration, you need to provide BackupConfiguration name and destination namespace. When we run the command the coping process consists of the following steps:
  
- At first, the cli copies Secret from source namespace to destination namespace.
- Then it copies Repository from source namespace to destination namespace.
- finally it copies BackupConfiguration from source namespace to destination namespace.

You will provide the destination namespace by using flags. The available flags are:

| Flag             | Description                                                                         |
|------------------|-------------------------------------------------------------------------------------|
| `--namespace`    | Indicates the namespace of the respective `BackupConfiguration`.                    |
| `--to-namespace` | Indicates the destination namespace where the `BackupConfiguration` will be copied. |

**Format:**

```bash
kubectl stash cp backupconfig <backupconfig-name> [flags]
```

**Example:**

```bash
$ kubectl stash cp backupconfig my-backupconfig --namespace=demo --to-namespace=demo1
```

### Copy VolumeSnapshot

To copy a VolumeSnapshot, you need to provide `VolumeSnapshot` name and destination namespace. You will provide the destination namespace by using flag. The available flags are:

| Flag             | Description                                                                    |
|------------------|--------------------------------------------------------------------------------|
| `--namespace`    | Indicates the namespace of the respective `VolumeSnapshot`.                    |
| `--to-namespace` | Indicates the destination namespace where the `VolumeSnapshot` will be copied. |

**Example:**

```bash
$ kubectl stash cp volumesnapshot my-vol-snap --namespace=demo --to-namespace=demo1
```

## Clone PVC

`kubectl stash clone pvc` command is used to clone PVC from one namespace to another namespace. When we run the command the cloning process consists of the following steps:

- At first, It creates a Repository in the source namespace.
- Using this repository, it creates a BackupConfiguration targeting the PVC to take backup.
- After the backup process succeeded, It copies Repository to the destination namespace
- finally, It restores the backed up data into VolumeClaimTemplate in the destination namespace.

To clone a PVC, you need to provide backend credentials for creating Repository.
You will provide the backend credential by using flags. The available flags are:

| Flag                | Description                                                                   |
|---------------------|-------------------------------------------------------------------------------|
| `--namespace`       | Indicates namespace of the respective pvc.                                    |
| `--to-namespace`    | Indicates the destination namespace where the PVC will be cloned.             |
| `--secret`          | Specify the name of the storage secret that will be used to create Repository |
| `--bucket`          | Specify the name of the cloud bucket/container.                               |
| `--prefix`          | Prefix denotes the directory inside the backend.                              |
| `--provider`        | Specify backend provider (i.e. gcs, s3, azure etc)                            |
| `--endpoint`        | Endpoint for s3/s3 compatible backend                                         |
| `--max-connections` | Specify maximum concurrent connections for GCS, Azure and B2 backend.         |

**Format:**

```bash
kubectl stash clone pvc <pvc-name> [flags]
```

**Example:**

```bash
$ kubectl stash clone pvc my-pvc -n demo --to-namespace=demo-1 --secret=<secret> --bucket=<bucket> --prefix=<prefix> --provider=<provider>
```

## Download Snapshots

`kubectl stash download` command is used to download the snapshots from backend repository into your local machine.
To download the snapshots you have to provide `Repository` name, download directory and snapshot list. You will provide the download directory and snapshot list using flags. The available flags are:

| Flag            | Description                                                           |
|-----------------|-----------------------------------------------------------------------|
| `--namespace`   | Indicates the namespace of the respective `Repository`.               |
| `--destination` | Indicates the local directory where the snapshots will be downloaded. |                                     |
| `--snapshots`   | Specifies the list of snapshots. (Provide a list of snapshot ID)      |

**Format:**

```bash
kubectl stash download <repository-name> [flags]
```

**Example:**

```bash
$ kubectl stash download gcs-repo --namespace=demo --destination=/home/downloads/ --snapshots="19eb4793,b7244d52"

```
Use `kubectl get snapshots` to get the available snapshots.

## Trigger an Instant Backup

`kubectl stash trigger` command is used to take an instant backup in stash.
To trigger an instant backup, you need to have a BackupConfiguration in your cluster. You need to provide the BackupConfiguration name. You can also provide the namespace by using the `--namespace` flag. This flag indicates the namespace where the trigger backup will be occurred.

**Format:**

```bash
$ kubectl stash trigger <backupconfig-name>[flags]
```

**Example:**

```bash
$ kubectl stash trigger my-config --namespace=demo
```

## Unlock Repository

`kubectl stash unlock` are used to remove lock from the backend repository.
To unlock the Repository, you need to provide a Repository name. You can also provide the namespace by using the `--namespace` flag. This flag indicates the Repository namespace.

**Format:**

```bash
kubectl stash unlock <repository-name> [flags]
```

**Example:**

```bash
$ kubectl stash unlock my-repo --namespace=demo
```

## Pause Backup
`kubectl stash pause` command is used to pause Stash backup temporarily. To pause a backup you have to provide the `BackupConfiguration` name or `BackupBatch` name by using flags. The available flags are:

| Flag             | Description                                                                       |
|------------------|-----------------------------------------------------------------------------------|
| `--namespace`    | Indicates the namespace of the respective `BackupConfiguration` or `BackupBatch`. |
| `--backupconfig` | Name of the `BackupConfiguration`.                                                |
| `--backupbatch`  | Name of the `BackupBatch`.                                                        |

**Format:**

```bash
kubectl stash pause backup [flags]
```

**Example:**

```bash
$ kubectl stash pause backup --namespace=demo --backupconfig=my-config
```

## Resume Backup

`kubectl stash pause` comand is used to resume a backup. To resume a backup you have to provide the `BackupConfiguration` name or `BackupBatch` name by using flags. The available flags are:

| Flag             | Description                                                                       |
|------------------|-----------------------------------------------------------------------------------|
| `--namespace`    | Indicates the namespace of the respective `BackupConfiguration` or `BackupBatch`. |
| `--backupconfig` | Name of the `BackupConfiguration`.                                                |
| `--backupbatch`  | Name of the `BackupBatch`.                                                        |

**Format:**

```bash
kubectl stash resume backup [flags]
```

**Example:**

```bash
$ kubectl stash resume backup --namespace=demo --backupconfig==my-config
```

## Debug Command
`kubectl stash debug` command is used to debug stash resources. This command describes the necessary resources and shows logs from the related pods which makes the debugging process quicker and easier.

### Debug Backup
This command is used to debug a backup. You have to provide the BackupConfiguration name or BackupBatch name by using flags. The available flags are:

| Flag             | Description                                                                       |
|------------------|-----------------------------------------------------------------------------------|
| `--namespace`    | Indicates the namespace of the respective `BackupConfiguration` or `BackupBatch`. |
| `--backupconfig` | Name of the `BackupConfiguration`.                                                |
| `--backupbatch`  | Name of the `BackupBatch`.                                                        |


**Format:**

```bash
kubectl stash debug backup [flags]
```

**Example:**

```bash
$ kubectl stash debug backup --namespace=demo --backupconfig=my-config
```

### Debug Restore

This command is used to debug a restore. You have to provide the `RestoreSession` name or `RestoreBatch` name by using flags. The available flags are:

| Flag               | Description                                                                   |
|--------------------|-------------------------------------------------------------------------------|
| `--namespace`      | Indicates the namespace of the respective `RestoreSession` or `RestoreBatch`. |
| `--restoresession` | Name of the `RestoreSession`.                                                 |
| `--restorebatch`   | Name of the `RestoreBatch`.                                                   |


**Format:**

```bash
kubectl stash debug restore [flags]
```

**Example:**

```bash
$ kubectl stash debug restore --namespace=demo --restoresession=my-restore 
```

### Debug Operator

This command is used to show version related information and operator logs.

**Example:**

```bash
$ kubectl stash debug operator
```

## Key Command

`kubectl stash key` command is used to manage keys (passwords) of a restic repository.

### Key List

This command is used to list restic keys (passwords) of a restic repository.

**Format:**

```bash
kubectl stash key list <repository-name> [flags]
```

**Example:**

```bash
$ kubectl stash key list my-repo --namespace=demo
```

### Key Add

This command is used to add a new key (password) to a restic repository. You have to provide the information of the new key by using flags. The available flags are:

| Flag                  | Description                                             |
|-----------------------|---------------------------------------------------------|
| `--namespace`         | Indicates the namespace of the respective `Repository`. |
| `--host`              | Host of the new key.                                    |
| `--user`              | User of the new key.                                    |
| `--new-password-file` | File from which to read the new password file.          |

**Format:**

```bash
kubectl stash key add <repository-name> [flags]
```

**Example:**

```bash
$ kubectl stash key list my-repo --namespace=demo --user root --host my-host --new-password-file password.txt
```

### Key Update

This command is used to update the current key (password) of a restic repository. The available flags for this command are:

| Flag                  | Description                                             |
|-----------------------|---------------------------------------------------------|
| `--namespace`         | Indicates the namespace of the respective `Repository`. |
| `--new-password-file` | File from which to read the new password file.          |

**Format:**

```bash
kubectl stash key update <repository-name> [flags]
```

**Example:**

```bash
$ kubectl stash key list my-repo --namespace=demo --new-password-file password.txt
```

### Key Remove

This command is used to remove a key (password) of a restic repository. The available flags for this command are:

| Flag          | Description                                             |
|---------------|---------------------------------------------------------|
| `--namespace` | Indicates the namespace of the respective `Repository`. |
| `--id`        | ID of the restic key.                                   |

**Format:**

```bash
kubectl stash key remove <repository-name> [flags]
```

**Example:**

```bash
$ kubectl stash key remove my-repo --namespace=demo --id cdc89a7d
```

## Generate Rules

`kubectl stash gen rules` command is used to create rules for a RestoreSession to recover the database and application 
backups. This command finds the nearest repository snapshots for a given timestamp and generates two rules: one for the snapshots just before the specified timestamp and another for those at or after the specified timestamp. The available flags for this command are:

| Flag                 | Description                                                 |
|----------------------|-------------------------------------------------------------|
| `--namespace`        | Indicates the namespace of the respective `Repository`.     |
| `--timestamp`        | Timestamp to find the closest snapshots.                    |
| `--group-interval`   | Specifies the time gap between batches of backup snapshots. |
| `--request-timeout`  | Request timeout duration for the kubectl command.           |  

**Format:**

```bash
kubectl stash gen rules <repository-name> [flags]
```
**Example:**

```bash
$ kubectl stash gen rules my-repo --namespace=demo --timestamp 2023-10-05T05:16:11Z
```

## Check Repository

`kubectl stash check` command is used to test the restic repository for errors and reports any errors it finds. It can also be used to read all data and therefore simulate a restore. By default, the command will always load all data directly from the repository and not use a local cache. The available flags for this command are:

| Flag                 | Description                                                                                                                                                         |
|----------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `--namespace`        | Indicates the namespace of the respective `Repository`.                                                                                                             |
| `--read-data`        | Read all data blobs.                                                                                                                                                |
| `--with-cache`       | Use existing cache, only read uncached data from repository.                                                                                                        |
| `--read-data-subset` | Read a subset of data packs, specified as 'n/t' for specific part, or either 'x%' or 'x.y%' or a size in bytes with suffixes k/K, m/M, g/G, t/T for a random subset |  

**Format**

```bash
kubectl stash check <repository-name> [flags]
```

**Example**

```bash
$ kubectl stash check my-repo --namespace=demo --read-data
```

## Rebuild Index

`kubectl stash rebuild-index` command is used to create a new index based on the pack files in the restic repository. 

| Flag               | Description                                             |
|--------------------|---------------------------------------------------------|
| `--namespace`      | Indicates the namespace of the respective `Repository`. |
| `--read-all-packs` | Read all pack files to generate new index from scratch. |

**Format**

```bash
kubectl stash rebuild-index <repository-name> [flags]
```

**Example**

```bash
$ kubectl stash rebuild-index my-repo --namespace=demo --read-all-packs
```

## Migrate Repository

`kubectl stash migrate` command is used to check the repository integrity and then upgrade the repository version to 2. After the migration is complete, run `kubectl stash prune` to compress the repository metadata.

| Flag          | Description                                             |
|---------------|---------------------------------------------------------|
| `--namespace` | Indicates the namespace of the respective `Repository`. |

**Format**

```bash
kubectl stash migrate <repository-name> [flags]
```

**Example**

```bash
$ kubectl stash migrate my-repo --namespace=demo
```

## Prune

`kubectl stash prune` command is used to remove unneeded data from the repository. 

| Flag                      | Description                                                                                                                                         |
|---------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| `--namespace`             | Indicates the namespace of the respective `Repository`.                                                                                             |
| `--max-unused-limit`      | Tolerate given limit of unused data (absolute value in bytes with suffixes k/K, m/M, g/G, t/T, a value in % or the word 'unlimited') (default `5%`) |
| `--max-repack-size`       | Maximum size to repack (allowed suffixes: k/K, m/M, g/G, t/T)                                                                                       |
| `--dry-run`               | Do not modify the repository, just print what would be done                                                                                         |
| `--repack-uncompressed`   | Repack all uncompressed data                                                                                                                        |
| `--repack-cacheable-only` | Only repack packs which are cacheable                                                                                                               |
| `--repack-small`          | Repack pack files below `80%` of target pack size                                                                                                   |

**Format**

```bash
kubectl stash prune <repository-name> [flags]
```

**Example**

```bash
$ kubectl stash prune my-repo --namespace=demo
```
