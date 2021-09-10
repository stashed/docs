---
title: NATS Backup Customization
description: Customizing NATS Backup and Restore process with Stash
menu:
  docs_{{ .version }}:
    identifier: stash-nats-customization
    name: Customizing Backup & Restore Process
    parent: stash-nats
    weight: 40
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: stash-addons
---

# Customizing Backup and Restore Process

Stash provides rich customization supports for the backup and restore process to meet the requirements of various cluster configurations. This guide will show you some examples of these customizations.

> Note: YAML files used in this tutorial are stored [here](https://github.com/stashed/docs/tree/{{< param "info.version" >}}/docs/addons/nats/customization/examples).

## Customizing Backup Process

In this section, we are going to show you how to customize the backup process. Here, we are going to show some examples of providing arguments to the backup process, running the backup process as a specific user, ignoring some indexes during the backup process, etc.

### Backup specific streams

By default stash will take backup of all the streams. If you want to take backup of specific streams, you can pass a list of streams through `streams` param under `task.params` section.

The below example shows how you can pass the `"str1, str2"` to take backup of specific streams `str1` and ` str2`.

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: sample-nats-backup
  namespace: demo
spec:
  schedule: "*/2 * * * *"
  task:
    name: nats-backup-2.4.0
    params:
    - name: streams
      value: "str1, str2"
  repository:
    name: gcs-repo
  target:
    ref:
      apiVersion: appcatalog.appscode.com/v1alpha1
      kind: AppBinding
      name: sample-nats
  retentionPolicy:
    name: keep-last-5
    keepLast: 5
    prune: true
```

### Running backup job as a specific user

If your cluster requires running the backup job as a specific user, you can provide `securityContext` under `runtimeSettings.pod` section. The below example shows how you can run the backup job as the root user.

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: sample-nats-backup
  namespace: demo
spec:
  schedule: "*/2 * * * *"
  task:
    name: nats-backup-2.4.0
  repository:
    name: gcs-repo
  target:
    ref:
      apiVersion: appcatalog.appscode.com/v1alpha1
      kind: AppBinding
      name: sample-nats
  runtimeSettings:
    pod:
      securityContext:
        runAsUser: 0
        runAsGroup: 0
  retentionPolicy:
    name: keep-last-5
    keepLast: 5
    prune: true
```

### Specifying Memory/CPU limit/request for the backup job

If you want to specify the Memory/CPU limit/request for your backup job, you can specify `resources` field under `runtimeSettings.container` section.

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: sample-nats-backup
  namespace: demo
spec:
  schedule: "*/2 * * * *"
  task:
    name: nats-backup-2.4.0
  repository:
    name: gcs-repo
  target:
    ref:
      apiVersion: appcatalog.appscode.com/v1alpha1
      kind: AppBinding
      name: sample-nats
  runtimeSettings:
    container:
      resources:
        requests:
          cpu: "200m"
          memory: "1Gi"
        limits:
          cpu: "200m"
          memory: "1Gi"
  retentionPolicy:
    name: keep-last-5
    keepLast: 5
    prune: true
```

### Using multiple retention policies

You can also specify multiple retention policies for your backed up data. For example, you may want to keep few daily snapshots, few weekly snapshots, and few monthly snapshots, etc. You just need to pass the desired number with the respective key under the `retentionPolicy` section.

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: BackupConfiguration
metadata:
  name: sample-nats-backup
  namespace: demo
spec:
  schedule: "*/2 * * * *"
  task:
    name: nats-backup-2.4.0
  repository:
    name: gcs-repo
  target:
    ref:
      apiVersion: appcatalog.appscode.com/v1alpha1
      kind: AppBinding
      name: sample-nat
  retentionPolicy:
    name: sample-nats-retention
    keepLast: 5
    keepDaily: 10
    keepWeekly: 20
    keepMonthly: 50
    keepYearly: 100
    prune: true
```

To know more about the available options for retention policies, please visit [here](/docs/concepts/crds/backupconfiguration.md#specretentionpolicy).

## Customizing Restore Process

In this section, we are going to show how you can overwrite existing streams, restore a specific snapshot, run restore job as a specific user, etc.

### Overwrite existing streams

By default stash will not overwrite any existing stream during the restore process. If you want to overwrite the existing streams with new ones,  you can pass `true` to the `overwrite` params under `task.params` section. 

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: sample-nats-restore
  namespace: demo
spec:
  task:
    name:nats-backup-2.4.0
    params:
    - name: overwrite
      value: true
  repository:
    name: gcs-repo
  target:
    ref:
      apiVersion: appcatalog.appscode.com/v1alpha1
      kind: AppBinding
      name: sample-nats
  rules:
  - snapshots: [latest]
```

### Restore specific snapshot

You can also restore a specific snapshot. At first, list the available snapshot as bellow,

```bash
❯ kubectl get snapshots -n demo
NAME                REPOSITORY   HOSTNAME   CREATED AT
gcs-repo-4bc21d6f   gcs-repo     host-0     2021-02-12T14:54:27Z
gcs-repo-f0ac7cbd   gcs-repo     host-0     2021-02-12T14:56:26Z
gcs-repo-9210ebb6   gcs-repo     host-0     2021-02-12T14:58:27Z
gcs-repo-0aff8890   gcs-repo     host-0     2021-02-12T15:00:28Z
```

>You can also filter the snapshots as shown in the guide [here](https://stash.run/docs/latest/concepts/crds/snapshot/#working-with-snapshot).

Stash adds the Repository name as a prefix of the Snapshot. You have to remove the repository prefix and use only the last 8 characters as the snapshot name during restore.

The below example shows how you can pass a specific snapshot name through the `snapshots` filed of `rules` section.

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: sample-nats-restore
  namespace: demo
spec:
  task:
    name: nats-backup-2.4.0
  repository:
    name: gcs-repo
  target:
    ref:
      apiVersion: appcatalog.appscode.com/v1alpha1
      kind: AppBinding
      name: sample-nats
  rules:
  - snapshots: [4bc21d6f]
```

>Please, do not specify multiple snapshots here. Each snapshot represents a complete backup of your database. Multiple snapshots are only usable during file/directory restore.

### Running restore job as a specific user

You can provide `securityContext` under `runtimeSettings.pod` section to run the restore job as a specific user.

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: sample-nats-restore
  namespace: demo
spec:
  task:
    name: nats-backup-2.4.0
  repository:
    name: gcs-repo
  target:
    ref:
      apiVersion: appcatalog.appscode.com/v1alpha1
      kind: AppBinding
      name: sample-nats
  runtimeSettings:
    pod:
      securityContext:
        runAsUser: 0
        runAsGroup: 0
  rules:
  - snapshots: [latest]
```

### Specifying Memory/CPU limit/request for the restore job

Similar to the backup process, you can also provide `resources` field under the `runtimeSettings.container` section to limit the Memory/CPU for your restore job.

```yaml
apiVersion: stash.appscode.com/v1beta1
kind: RestoreSession
metadata:
  name: sample-nats-restore
  namespace: demo
spec:
  task:
    name: nats-backup-2.4.0
  repository:
    name: gcs-repo
  target:
    ref:
      apiVersion: appcatalog.appscode.com/v1alpha1
      kind: AppBinding
      name: sample-nats
  runtimeSettings:
    container:
      resources:
        requests:
          cpu: "200m"
          memory: "1Gi"
        limits:
          cpu: "200m"
          memory: "1Gi"
  rules:
  - snapshots: [latest]
```
