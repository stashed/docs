---
title: BackupBatch Overview
menu:
  product_stash_{{ .version }}:
    identifier: backupbatch-overview
    name: BackupBatch
    parent: crds
    weight: 15
product_name: stash
menu_name: product_stash_{{ .version }}
section_menu_id: concepts
---

> New to Stash? Please start [here](/docs/concepts/README.md).

# BackupBatch

## What is BackupBatch

Sometimes, a single component may not meet the requirement for your application. For example, in order to deploy a WordPress, you will need a Deployment for the WordPress and another Deployment for database to store it's contents. Now, you may want to backup both of the deployment and database under a single configuration as they are parts of a single application.

A `BackupBatch` is a Kubernetes `CustomResourceDefinition`(CRD) which let you configure backup for multiple co-related components(workload, database etc.) under a single configuration.

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
    name: gcs-repo
  schedule: "*/3 * * * *"
  members:
  - target:
      ref:
        apiVersion: apps/v1
        kind: AppBinding
        name: sample-mysql
    task:
      name: mysql-backup-8.0.14
  - target:
      ref:
        apiVersion: apps/v1
        kind: Deployment
        name: wordpress
      volumeMounts:
        - name: wordpress-persistent-storage
          mountPath: /var/www/html
      paths:
        - /var/www/html
  retentionPolicy:
    name: 'keep-last-10'
    keepLast: 10
    prune: true
```

Here, we are going to describe the various sections of `BackupBatch` crd.

### BackupBatch `Spec`

A `BackupBatch` object has the following fields in the `spec` section.

#### spec.driver

`spec.driver` indicates the mechanism used to backup. Currently, Stash supports `Restic` and `VolumeSnapshotter` as drivers. The default value of this field is `Restic`. For more details, please see [here](/docs/concepts/crds/backupconfiguration.md#specdriver).

#### spec.members

`spec.members` field specifies a list of targets to backup. Each member consists of the following fields:

- **target.ref :** `target.ref` refers to the target of backup. You have to specify `apiVersion`, `kind` and `name` of the target. Stash will use this information to inject a sidecar to the target or to create a backup job for it.

- **target.paths :** `target.paths` specifies list of file paths to backup.

- **target.volumeMounts :** `target.volumeMounts` are the list of volumes and their `mountPath`s that contain the target file paths. Stash will mount these volumes inside a sidecar container or a backup job.

- **target.snapshotClassName :** `target.snapshotClassName` indicates the [VolumeSnapshotClass](https://kubernetes.io/docs/concepts/storage/volume-snapshot-classes/) to use for volume snasphotting. Use this field only if `driver` is set to `VolumeSnapshotter`.

- **task :** `task` specifies the name and parameters of the [Task](/docs/concepts/crds/task.md) crd to use to backup the target. For more details, please see [here](/docs/concepts/crds/backupconfiguration.md#spectask).

- **runtimeSettings :** `runtimeSettings` allows to configure runtime environment for the backup sidecar or job. You can specify runtime settings at both pod level and container level. For more details, please see [here](/docs/concepts/crds/backupconfiguration.md#specruntimesettings).

- **tempDir :** Stash mounts an `emptyDir` for holding temporary files. It is also used for `caching` for faster backup performance. You can configure the `emptyDir` using `tempDir` section. You can also disable `caching` using this field. For more details, please see [here](/docs/concepts/crds/backupconfiguration.md#spectempdir).

- **interimVolumeTemplate :** For some targets (i.e. some databases), Stash can't directly pipe the dumped data to the uploading process. In this case, it has to store the dumped data temporarily before uploading to the backend. `interimVolumeTemplate` specifies a PVC template for holding those  data temporarily. Stash will create a PVC according to the template and use it to store the data temporarily. This PVC will be deleted according to the [backupHistoryLimit](#specbackuphistorylimit). For more details, please see [here](/docs/concepts/crds/backupconfiguration.md#specinterimvolumetemplate).

#### spec.runtimeSettings

`spec.runtimeSettings` This runtime settings is applicable for CronJob(used to create `BackupSession`) only. For more details, please see [here](/docs/concepts/crds/backupconfiguration.md#specruntimesettings).

#### spec.repository

`spec.repository.name` indicates the `Repository` crd name that holds necessary backend information where the backed up data will be stored.

#### spec.schedule

`spec.schedule` is a [cron expression](https://en.wikipedia.org/wiki/Cron) that specifies the schedule of backup. Stash creates a Kubernetes [CronJob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/) with this schedule.

#### spec.backupHistoryLimit

`spec.backupHistoryLimit` specifies the number of `BackupSession` and its associate resources (Job, PVC etc.) to keep for debugging purposes. The default value of this field is 1. Stash will cleanup the old `BackupSession` and it's associate resources after each backup session according to `backupHistoryLimit`.

#### spec.paused

`spec.paused` can be used as `enable/disable` switch for backup. If it is set `true`, Stash will not take any backup of the target specified by this BackupBatch.

#### spec.retentionPolicy

`spec.retentionPolicy` specifies the policy to follow for cleaning old snapshots. For more details, please see [here](/docs/concepts/crds/backupconfiguration.md#specretentionpolicy).

### BackupBatch `Status`

A `BackupBatch` object has only a `observedGeneration` field in the `status` section.

- **observedGeneration :** The most recent generation observed by the `BackupBatch` controller.

## Next Steps

- Learn how to configure `BackupBatch` to backup data from [here](/docs/guides/latest/batch-backup/overview.md).
