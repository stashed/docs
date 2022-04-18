---
title: How to Troubleshoot Stash issues | Stash
description: Troubleshooting backup/restore issues
menu:
  docs_{{ .version }}:
    identifier: troubleshooting-how-to
    name: How to Troubleshoot
    parent: troubleshooting
    weight: 5
product_name: stash
menu_name: docs_{{ .version }}
section_menu_id: guides
---

# How to Troubleshoot Stash Issues

This guide will give you an overview of how you can gather the necessary information to identify the issue that causes the backup/restore failure.

## Troubleshoot Backup Issues

In this section, we are going to explain how to troubleshoot backup issues.

### Backup was Never Triggered

If you have created the desired `BackupConfiguration` but the respective backup triggering CronJob was never created or any `BackupSession` was not created in the scheduled time, in this case, follow the following steps:

#### Describe the `BackupConfiguration`

At first, you should describe the respective `BackupConfiguration` as below:

```bash
kubectl describe backupconfiguration <backupconfiguration name> -n <namespace>
```

Now, check the `Status` section of `BackupConfiguration`. Make sure all the `conditions` are `True`. If there is any issue during backup setup, you should see the error in the respective condition.

Also, check the event to see if there is any indication of an error.

#### Check Stash operator log

If you don't notice any error in the previous step, you should check the Stash operator log.

Run the following command to view the Stash operator log:

```bash
# Identify the Stash operator pod
kubectl get pods --all-namespaces | grep stash

# View log of Stash operator pod
kubectl logs -n <operator namespace> <stash operator pod name> -c operator
```

Now, inspect the log carefully. You should see the respective error in the log.

### BackupConfiguration NotReady

If the phase of the `BackupConfiguration` is `NotReady`, you should describe the respective `BackupConfiguration`,

```bash
kubectl describe backupconfiguration <backupconfiguration name> -n <namespace>
```

Now, check the `Status` section of `BackupConfiguration`. Make sure all the `conditions` are `True`. If there is any issue during backup setup, you should see the error in the respective condition. If any of the conditions is `False`, the backup stays in the `NotReady` phase. The conditions that cause a `BackupConfiguration` to become `NotReady` are given bellow:

| Reason | Message |
| ------------- | ------------- |
| RepositoryNotAvailable | Repository does not exist |
| BackendSecretNotAvailable | Backend Secret does not exist |
| TargetNotAvailable | Backup target does not exist |

### Backup Failed

If a backup fails, follow the following steps to identify the root cause.

#### Describe the `BackupSession`

At first, describe the failing `BackupSession`. You should see the respective error in the `Status` section.

```bash
kubectl describe backupsession <backupsession name> -n <namespace>
```

Also, check the `Events` section. Sometimes, it can be helpful to identify the issue.

#### View Backup Job/Sidecar log

If you don't see any error in the previous step, you should try checking the log of the respective backup job/sidecar.

If you are trying to backup a workload, run the following command to inspect the log:

```bash
kubectl logs -n <namespace> <workload pod name> -c stash
```

If you are trying to backup using Stash addons, run the following command to inspect the log:

```bash
# Identify the backup pod
kubectl get pods -n <namespace> | grep stash-backup

# View the log of the backup pod
kubectl logs -n <namespace> <backup pod name> --all-containers
```

Inspect the log carefully. You should notice the respective error that leads to backup failure.

## Troubleshoot Restore Issues

In this section, we are going to explain how to troubleshoot backup issues.

### Restore Pending

If the phase of the `RestoreSession` is `pending`, you should describe the respective `RestoreSession`,

```bash
kubectl describe restoresession <restoresession name> -n <namespace>
```

Now, check the `Status` section of `RestoreSession`. Make sure all the `conditions` are `True`. If there is any issue during backup setup, you should see the error in the respective condition. If any of the conditions is `False`, the `RestoreSession` stays in the `Pending` phase. The conditions that cause a `RestoreSession` to stay in `Pending` phase are given bellow:

| Reason | Message |
| ------------- | ------------- |
| RepositoryNotAvailable | Repository does not exist |
| BackendSecretNotAvailable | Backend Secret does not exist |
| TargetNotAvailable | Backup target does not exist |

### Restore Failed

If a restore fails, follow the following steps to identify the root cause.

#### Describe the `RestoreSession`

At first, describe the failing `RestoreSession`. You should see the respective error in the `Status` section.

```bash
kubectl describe restoresession <restoresession name> -n <namespace>
```

Also, check the `Events` section. Sometimes, it can be helpful to identify the issue.

#### View Restore Job/Init-Container log

If you don't see any error in the previous step, you should try checking the log of the respective restore job / init-container.

If you are trying to restore a workload, run the following command to inspect the log:

```bash
kubectl logs -n <namespace> <workload pod name> -c stash-init
```

If you are trying to restore using Stash addons, run the following command to inspect the log:

```bash
# Identify the restore pod
kubectl get pods -n <namespace> | grep stash-restore

# View the log of the restore pod
kubectl logs -n <namespace> <restore pod name> --all-containers
```

Inspect the log carefully. You should notice the respective error that leads to restore failure.

## Debug with Stash `kubectl` Plugin

You can use the Stash `kubectl` plugin to debug any backup/restore issue.

### Debug Backup

Run the following command to inspect any backup issue,

```bash
kubectl stash debug backup -n <namespace> --backupconfig <backupconfiguration name>
```

The above command will:

- Show version information of Kubernetes and Stash
- Describe relevent `BackupConfiguration`, `BackupSessions`, and `Pods`
- Show logs from the relevent `Pods`

### Debug Restore

Run the following command to inspect any restore issue,

```bash
kubectl stash debug restore -n <namespace> --restoresession <restoresession name>
```

The above command will:

- Show version information of Kubernetes and Stash
- Describe relevent `RestoreSession` and `Pods`
- Show logs from the relevent `Pods`

### Debug Operator

Run the following command to view the Stash operator log:

```bash
kubectl stash debug operator
```
