---
title: BackupConfiguration Overview
menu:
  docs_{{ .version }}:
    identifier: backupconfiguration-overview
    name: BackupConfiguration
    parent: crds
    weight: 10
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: concepts
---

> New to Stash? Please start [here](/docs/concepts/README.md).

# BackupConfiguration

## What is BackupConfiguration

A `BackupConfiguration` is a Kubernetes `CustomResourceDefinition`(CRD) which specifies the backup target, parameters(schedule, retention policy etc.) and a `Repository` object that holds snapshot storage information in a Kubernetes native way.

You have to create a `BackupConfiguration` object for each backup target. A backup target can be a workload, database or a PV/PVC.

## BackupConfiguration CRD Specification

Like any official Kubernetes resource, a `BackupConfiguration` has `TypeMeta`, `ObjectMeta` and `Spec` sections. However, unlike other Kubernetes resources, it does not have a `Status` section.

A sample `BackupConfiguration` object to backup the volumes of a Deployment is shown below:

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: demo-backup
  namespace: demo
spec:
  driver: Restic
  repository:
    name: local-repo
  # task:
  #   name: workload-backup # task field is not required for workload data backup but it is necessary for database backup.
  schedule: "* * * * *" # backup at every minutes
  paused: false
  backupHistoryLimit: 3
  target:
    alias: app-data
    ref:
      apiVersion: apps/v1
      kind: Deployment
      name: stash-demo
    paths:
    - /source/data
    exclude:
    - /source/data/not-important.txt
    - /source/data/*.html
    - /source/data/tmp/*
    volumeMounts:
    - name: source-data
      mountPath: /source/data
  hooks:
    preBackup:
      exec:
        command:
          - /bin/sh
          - -c
          - echo "Sample PreBackup hook demo"
      containerName: my-app-container
    postBackup:
      exec:
        command:
          - /bin/sh
          - -c
          - echo "Sample PostBackup hook demo"
      containerName: my-app-container
  runtimeSettings:
    container:
      resources:
        requests:
          memory: 256M
        limits:
          memory: 256M
      securityContext:
        runAsUser: 2000
        runAsGroup: 2000
      nice:
        adjustment: 5
      ionice:
        class: 2
        classData: 4
    pod:
      imagePullSecrets:
      - name:  my-private-registry-secret
      serviceAccountName: my-backup-svc
  tempDir:
    medium: "Memory"
    sizeLimit: "2Gi"
    disableCache: false
  retentionPolicy:
    name: 'keep-last-5'
    keepLast: 5
    prune: true
```

Here, we are going to describe the various sections of `BackupConfiguration` crd.

### BackupConfiguration `Spec`

A `BackupConfiguration` object has the following fields in the `spec` section.

#### spec.driver

`spec.driver` indicates the mechanism used to backup a target. Currently, Stash supports `Restic` and `VolumeSnapshotter` as drivers. The default value of this field is `Restic`.

| Driver              | Usage                                                                                                                                                                                                                    |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `Restic`            | Used to backup workload data, persistent volumes data and databases. It uses [restic](https://restic.net) to backup the target.                                                                                          |
| `VolumeSnapshotter` | Used to take snapshot of PersistentVolumeClaims of a targeted workload. It leverages Kubernetes [VolumeSnapshot](https://kubernetes.io/docs/concepts/storage/volume-snapshots/) crd and CSI driver to snapshot the PVCs. |

#### spec.target

`spec.target` field indicates the target for backup runs. This field consists of the following sub-fields:

- **spec.target.alias :** The alias is used as an identifer of the backed up data in the backend. This is particularly useful for `BackupBatch` where multiple targets are backed up into a single repository.

- **spec.target.ref :** `spec.target.ref` refers to the target of backup. You have to specify `apiVersion`, `kind` and `name` of the target. Stash will use this information to inject a sidecar to the target or to create a backup job for it.

- **spec.target.paths :** `spec.target.paths` specifies list of file paths to backup.

- **spec.target.exclude :** Specifies a list of pattern for the files that should be ignored during backup. Stash will not backup the files that matches these patterns.

- **spec.target.volumeMounts :** `spec.target.volumeMounts` are the list of volumes and their `mountPath`s that contain the target file paths. Stash will mount these volumes inside a sidecar container or a backup job.

- **spec.target.snapshotClassName:** `spec.target.snapshotClassName` indicates the [VolumeSnapshotClass](https://kubernetes.io/docs/concepts/storage/volume-snapshot-classes/) to use for volume snasphotting. Use this field only if `spec.driver` is set to `VolumeSnapshotter`.

#### spec.repository

`spec.repository.name` indicates the `Repository` crd name that holds necessary backend information where the backed up data will be stored.

#### spec.schedule

`spec.schedule` is a [cron expression](https://en.wikipedia.org/wiki/Cron) that specifies the schedule of backup. Stash creates a Kubernetes [CronJob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/) with this schedule.

#### spec.backupHistoryLimit

`spec.backupHistoryLimit` specifies the number of `BackupSession` and its associate resources (Job, PVC etc.) to keep for debugging purposes. The default value of this field is 1. Stash will cleanup the old `BackupSession` and it's associate resources after each backup session according to `backupHistoryLimit`.

#### spec.task

`spec.task` specifies the name and parameters of the [Task](/docs/concepts/crds/task.md) crd to use to backup the target.

- **spec.task.name:** `spec.task.name` indicates the name of the `Task` to use for this backup process.
- **spec.task.params:** `spec.task.params` is an array of custom parameters to use to configure the task.

> `spec.task` section is not required for backing up workload data (i.e. Deployment, DaemonSet, StatefulSet etc.). However, it is necessary for backing up databases and stand-alone PVCs.

#### spec.paused

`spec.paused` can be used as `enable/disable` switch for backup. If it is set `true`, Stash will not take any backup of the target specified by this BackupConfiguration.

#### spec.hooks

`spec.hooks` allows performing some actions before and after the backup process. You can send HTTP requests to a remote server via `httpGet` or `httpPost` hooks. You can check whether a TCP socket is open using `tcpSocket` hook. You can also execute some commands into your application pod using `exec` hook.

- **spec.hooks.preBackup:** `spec.hooks.preBackup` hooks are executed before the backup process.
- **spec.hooks.postBackup:** `spec.hooks.postBackup` hooks are executed after the backup process.

For more details on how hooks work in Stash and how to configure different types of hook, please visit [here](/docs/guides/latest/hooks/overview.md).

#### spec.runtimeSettings

`spec.runtimeSettings` allows to configure runtime environment for the backup sidecar or job. You can specify runtime settings at both pod level and container level.

- **spec.runtimeSettings.container**

  `spec.runtimeSettings.container` is used to configure the backup sidecar/job at container level. You can configure the following container level parameters:

| Field             | Usage                                                                                                                                                                                                                           |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `resources`       | Compute resources required by the sidecar container or backup job. To learn how to manage resources for containers, please visit [here](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/). |
| `livenessProbe`   | Periodic probe of backup sidecar/job container's liveness. Container will be restarted if the probe fails.                                                                                                                      |
| `readinessProbe`  | Periodic probe of backup sidecar/job container's readiness. Container will be removed from service endpoints if the probe fails.                                                                                                |
| `lifecycle`       | Actions that the management system should take in response to container lifecycle events.                                                                                                                                       |
| `securityContext` | Security options that backup sidecar/job's container should run with. For more details, please visit [here](https://kubernetes.io/docs/concepts/policy/security-context/).                                                      |
| `nice`            | Set CPU scheduling priority for backup process. For more details about `nice`, please visit [here](https://www.askapache.com/optimize/optimize-nice-ionice/#nice).                                                              |
| `ionice`          | Set I/O scheduling class and priority for backup process. For more details about `ionice`, please visit [here](https://www.askapache.com/optimize/optimize-nice-ionice/#ionice).                                                |
| `env`             | A list of the environment variables to set in the sidecar container or backup job's container.                                                                                                                                  |
| `envFrom`         | This allows to set environment variables to the container that will be created for this function from a Secret or ConfigMap.                                                                                                    |

- **spec.runtimeSettings.pod**

  `spec.runtimeSettings.pod` is used to configure backup job in pod level. You can configure the following pod level parameters,

| Field                          | Usage                                                                                                                                                                                                                                    |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `serviceAccountName`           | Name of the `ServiceAccount` to use for the backup job. Stash sidecar will use the same `ServiceAccount` as the target workload.                                                                                                         |
| `nodeSelector`                 | Selector which must be true for backup job pod to fit on a node.                                                                                                                                                                         |
| `automountServiceAccountToken` | Indicates whether a service account token should be automatically mounted into the backup pod.                                                                                                                                           |
| `nodeName`                     | `nodeName` is used to request to schedule backup job's pod onto a specific node.                                                                                                                                                         |
| `securityContext`              | Security options that backup job's pod should run with. For more details, please visit [here](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/).                                                               |
| `imagePullSecrets`             | A list of secret names in the same namespace that will be used to pull image from private Docker registry. For more details, please visit [here](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/). |
| `affinity`                     | Affinity and anti-affinity to schedule backup job's pod on a desired node. For more details, please visit [here](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity).                         |
| `schedulerName`                | Name of the scheduler that should dispatch the backup job.                                                                                                                                                                               |
| `tolerations`                  | Taints and Tolerations to ensure that backup job's pod is not scheduled in inappropriate nodes. For more details about `toleration`, please visit [here](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/).       |
| `priorityClassName`            | Indicates the backup job pod's priority class. For more details, please visit [here](https://kubernetes.io/docs/concepts/configuration/pod-priority-preemption/).                                                                        |
| `priority`                     | Indicates the backup job pod's priority value.                                                                                                                                                                                           |
| `readinessGates`               | Specifies additional conditions to be evaluated for Pod readiness. For more details, please visit [here](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-readiness-gate).                                          |
| `runtimeClassName`             | RuntimeClass is used for selecting the container runtime configuration. For more details, please visit [here](https://kubernetes.io/docs/concepts/containers/runtime-class/)                                                             |
| `enableServiceLinks`           | EnableServiceLinks indicates whether information about services should be injected into pod's environment variables.                                                                                                                     |

#### spec.tempDir

Stash mounts an `emptyDir` for holding temporary files. It is also used for `caching` for faster backup performance. You can configure the `emptyDir` using `spec.tempDir` section. You can also disable `caching` using this field. The following fields are configurable in `spec.tempDir` section:

- **spec.tempDir.medium :** Specifies the type of storage medium should back this directory.
- **spec.tempDir.sizeLimit :** Maximum limit of storage for this volume.
- **spec.tempDir.disableCaching :** Disable caching while backup. This may negatively impact backup performance. This is set to `false` by default.

#### spec.interimVolumeTemplate

For some targets (i.e. some databases), Stash can't directly pipe the dumped data to the uploading process. In this case, it has to store the dumped data temporarily before uploading to the backend. `spec.interimVolumeTemplate` specifies a PVC template for holding those  data temporarily. Stash will create a PVC according to the template and use it to store the data temporarily. This PVC will be deleted according to the [spec.backupHistoryLimit](#specbackuphistorylimit).

>Note that the usage of this field is different than `spec.tempDir` which is used for caching purpose. Stash has introduced this field because the `emptyDir` volume that is used for `spec.tempDir` does not play nice with large databases( i.e. 100Gi database). Also, it provides debugging capability as Stash keeps it until it hits the limit specified in `spec.backupHistoryLimit`.

#### spec.retentionPolicy

`spec.retentionPolicy` specifies the policy to follow for cleaning old snapshots. Following options are available to configure retention policy:

| Policy        | Value   | `restic` forget command flag | Description                                                                                        |
| ------------- | ------- | ---------------------------- | -------------------------------------------------------------------------------------------------- |
| `name`        | string  |                              | Name of retention policy. You can provide any name.                                                |
| `keepLast`    | integer | --keep-last n                | Never delete the **n** last (most recent) snapshots.                                               |
| `keepHourly`  | integer | --keep-hourly n              | For the last **n** hours in which a snapshot was made, keep only the last snapshot for each hour.  |
| `keepDaily`   | integer | --keep-daily n               | For the last **n** days which have one or more snapshots, only keep the last one for that day.     |
| `keepWeekly`  | integer | --keep-weekly n              | For the last **n** weeks which have one or more snapshots, only keep the last one for that week.   |
| `keepMonthly` | integer | --keep-monthly n             | For the last **n** months which have one or more snapshots, only keep the last one for that month. |
| `keepYearly`  | integer | --keep-yearly n              | For the last **n** years which have one or more snapshots, only keep the last one for that year.   |
| `keepTags`    | array   | --keep-tag <tag>             | Keep all snapshots which have all tags specified by this option (can be specified multiple times). |
| `prune`       | bool    | --prune                      | If set `true`, Stash will cleanup unreferenced data from the backend.                              |
| `dryRun`      | bool    | --dry-run                    | Stash will not remove anything but print which snapshots would be removed.                         |

### BackupConfiguration `Status`

A `BackupConfiguration` object has the following fields in the `status` section.

- **observedGeneration :** The most recent generation observed by the `BackupConfiguration` controller.

- **conditions :** The `spec.conditions` shows current backup setup condition for this BackupConfiguration. The following conditions are set by the Stash operator:

| Condition Type       | Usage                                                                |
| -------------------- | -------------------------------------------------------------------- |
| `RepositoryFound`    | Indicates whether the respective Repository object was found or not. |
| `BackendSecretFound` | Indicates whether the respective backend secret was found or not.    |
| `CronJobCreated`     | Indicates whether the backup triggering CronJob was created  or not. |

## Next Steps

- Learn how to configure `BackupConfiguration` to backup workloads data from [here](/docs/guides/latest/workloads/overview.md).
- Learn how to configure `BackupConfiguration` to backup databases from [here](/docs/guides/latest/addons/overview.md).
- Learn how to configure `BackupConfiguration` to backup stand-alone PVC from [here](/docs/guides/latest/volumes/overview.md).
