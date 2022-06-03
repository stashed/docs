---
title: BackupBatch Overview
menu:
  docs_{{ .version }}:
    identifier: backupbatch-overview
    name: BackupBatch
    parent: crds
    weight: 15
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: concepts
---

{{< notice type="warning" message="This is an Enterprise-only feature. Please install [Stash Enterprise Edition](/docs/setup/install/enterprise.md) to try this feature." >}}

# BackupBatch

## What is BackupBatch

Sometimes, a single component may not meet the requirement for your application. For example, in order to deploy a WordPress, you will need a Deployment for the WordPress and another Deployment for a database to store its contents. Now, you may want to backup both of the deployment and database together as they are parts of a single application.

A `BackupBatch` is a Kubernetes `CustomResourceDefinition`(CRD) which lets you configure backup for multiple co-related components(workload, database, etc.) together.

## BackupBatch CRD Specification

Like any official Kubernetes resource, a `BackupBatch` has `TypeMeta`, `ObjectMeta`, `Spec` and `Status` sections.

A sample `BackupBatch` object to backup multiple co-related components is shown below:

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupBatch
metadata:
  name: deploy-backup-batch
  namespace: demo
spec:
  repository:
    name: minio-repo
  schedule: "*/3 * * * *"
  executionOrder: Parallel
  members:
  - target:
      alias: db
      ref:
        apiVersion: apps/v1
        kind: AppBinding
        name: wordpress-mysql
    task:
      name: mysql-backup-8.0.14
  - target:
      alias: app
      ref:
        apiVersion: apps/v1
        kind: Deployment
        name: wordpress
      volumeMounts:
      - name: wordpress-persistent-storage
        mountPath: /var/www/html
      paths:
      - /var/www/html
      exclude:
      - /var/www/html/my-file.html
      - /var/www/html/*.json
  hooks:
    preBackup:
      exec:
        command:
          - /bin/sh
          - -c
          - echo "Sample PreBackup hook demo"
      containerName: my-database-container
    postBackup:
      exec:
        command:
          - /bin/sh
          - -c
          - echo "Sample PostBackup hook demo"
      containerName: my-database-container
  retentionPolicy:
    name: 'keep-last-10'
    keepLast: 10
    prune: true
  timeOut: 5m
```

Here, we are going to describe the various sections of `BackupBatch` crd.

### BackupBatch `Spec`

A `BackupBatch` object has the following fields in the `spec` section.

#### spec.driver

`spec.driver` indicates the mechanism used to backup. Currently, Stash supports `Restic` and `VolumeSnapshotter` as drivers. The default value of this field is `Restic`. For more details, please see [here](/docs/concepts/crds/backupconfiguration.md#specdriver).

#### spec.repository

`spec.repository.name` indicates the `Repository` crd name that holds necessary backend information where the backed up data will be stored.

#### spec.schedule

`spec.schedule` is a [cron expression](https://en.wikipedia.org/wiki/Cron) that specifies the schedule of backup. Stash creates a Kubernetes [CronJob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/) with this schedule.

#### spec.executionOrder

`spec.executionOrder` specifies whether Stash should backup the targets sequentially or parallelly. If `spec.executionOrder` is set to `Parallel`, Stash will start backup of all the targets simultaneously. If it is set to `Sequential`, Stash will not start backup of a target until all the previous members have completed their backup process. The default value of this field is `Parallel`.

#### spec.members

`spec.members` field specifies a list of targets to backup. Each member consists of the following fields:

- **target :** Each member has a target specification. The target specification of a member is the same as the target specification of a `BackupConfiguration` explained [here](/docs/concepts/crds/backupconfiguration.md#spectarget).

- **task :** `task` specifies the name and parameters of the [Task](/docs/concepts/crds/task.md) crd to use to backup the target. For more details, please see [here](/docs/concepts/crds/backupconfiguration.md#spectask).

- **runtimeSettings :** `runtimeSettings` allows to configure runtime environment for the backup sidecar or job. You can specify runtime settings at both pod level and container level. For more details, please see [here](/docs/concepts/crds/backupconfiguration.md#specruntimesettings).

- **tempDir :** Stash mounts an `emptyDir` for holding temporary files. It is also used for `caching` for faster backup performance. You can configure the `emptyDir` using `tempDir` section. You can also disable `caching` using this field. For more details, please see [here](/docs/concepts/crds/backupconfiguration.md#spectempdir).

- **interimVolumeTemplate :** For some targets (i.e. some databases), Stash can't directly pipe the dumped data to the uploading process. In this case, it has to store the dumped data temporarily before uploading to the backend. `interimVolumeTemplate` specifies a PVC template for holding those data temporarily. Stash will create a PVC according to the template and use it to store the data temporarily. This PVC will be deleted according to the [backupHistoryLimit](#specbackuphistorylimit). For more details, please see [here](/docs/concepts/crds/backupconfiguration.md#specinterimvolumetemplate).

- **hooks :** Each member has its own hook field which allows you to execute member-specific pre-backup or post-backup hooks. For more details about hooks, please visit [here](/docs/concepts/crds/backupconfiguration.md#spechooks).

#### spec.hooks

`spec.hooks` allows performing some global actions before and after the backup process of the members. You can send HTTP requests to a remote server via `httpGet` or `httpPost`. You can check whether a TCP port is open using `tcpSocket` hooks. You can also execute some commands using `exec` hook.

- **spec.hooks.preBackup:** `spec.hooks.preBackup` hooks are executed on each backup session before taking backup of any of the members.
- **spec.hooks.postBackup:** `spec.hooks.postBackup` hooks are executed on each backup session after taking backup of all the members.

For more details on how hooks work in Stash and how to configure different types of hook, please visit [here](/docs/guides/hooks/overview/index.md).

#### spec.runtimeSettings

`spec.runtimeSettings` This runtime settings is applicable for CronJob(used to create `BackupSession`) only. For more details, please see [here](/docs/concepts/crds/backupconfiguration.md#specruntimesettings).

#### spec.backupHistoryLimit

`spec.backupHistoryLimit` specifies the number of `BackupSession` and its associate resources (Job, PVC etc.) to keep for debugging purposes. The default value of this field is 1. Stash will clean up the old `BackupSession` and it's associate resources after each backup session according to `backupHistoryLimit`.

#### spec.paused

`spec.paused` can be used as `enable/disable` switch for backup. If it is set `true`, Stash will not take any backup of the target specified by this BackupBatch.

#### spec.retentionPolicy

`spec.retentionPolicy` specifies the policy to follow for cleaning old snapshots. For more details, please see [here](/docs/concepts/crds/backupconfiguration.md#specretentionpolicy).

#### spec.timeOut

`spec.timeOut` specifies the maximum duration of the backup. `BackupSession` will be considered `Failed` if the backup does not complete within this time limit. By default, Stash don't set any timeout for the backup.

### BackupBatch `Status`

A `BackupBatch` object has the following fields in the `status` section.

- **observedGeneration :** The most recent generation observed by the `BackupBatch` controller.

- **conditions :** The `status.conditions` shows current backup setup condition for this BackupBatch. The following conditions are set by the Stash operator:

| Condition Type       | Usage                                                                |
| -------------------- | -------------------------------------------------------------------- |
| `RepositoryFound`    | Indicates whether the respective Repository object was found or not. |
| `BackendSecretFound` | Indicates whether the respective backend secret was found or not.    |
| `CronJobCreated`     | Indicates whether the backup triggering CronJob was created  or not. |

- **memberConditions :** Shows current backup setup condition of the members of a `BackupBatch`. Each entry has the following two fields:
  - **target :** Points to the respective target whose condition is shown here.
  - **conditions:** Shows the current backup setup condition of this member.

The following conditions are set for the members of a `BackupBatch`.

| Condition Type         | Usage                                                                                                                                                    |
| ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `BackupTargetFound`    | Indicates whether the backup target was found or not.                                                                                                    |
| `StashSidecarInjected` | Indicates whether `stash` sidecar was injected into the targeted workload or not. This condition is set only for the target that uses the sidecar model. |

## Next Steps

- Learn how to configure `BackupBatch` to backup data from [here](/docs/guides/batch-backup/overview.md).
