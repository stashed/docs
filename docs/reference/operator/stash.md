---
title: Stash
menu:
  docs_{{ .version }}:
    identifier: stash
    name: Stash
    parent: reference-operator
    weight: 0

menu_name: docs_{{ .version }}
section_menu_id: reference
url: /docs/{{ .version }}/reference/operator/
aliases:
- /docs/{{ .version }}/reference/operator/stash/
---
## stash

Stash by AppsCode - Backup your Kubernetes Volumes

### Synopsis

Stash is a Kubernetes operator for restic. For more information, visit here: https://stash.run

### Options

```
      --bypass-validating-webhook-xray   if true, bypasses validating webhook xray checks
  -h, --help                             help for stash
      --use-kubeapiserver-fqdn-for-aks   if true, uses kube-apiserver FQDN for AKS cluster to workaround https://github.com/Azure/AKS/issues/522 (default true)
```

### SEE ALSO

* [stash backup-pvc](/docs/reference/operator/stash_backup-pvc.md)	 - Takes a backup of Persistent Volume Claim
* [stash create-backupsession](/docs/reference/operator/stash_create-backupsession.md)	 - create a BackupSession
* [stash create-vs](/docs/reference/operator/stash_create-vs.md)	 - Take snapshot of PersistentVolumeClaims
* [stash forget](/docs/reference/operator/stash_forget.md)	 - Delete snapshots from a restic repository
* [stash restore](/docs/reference/operator/stash_restore.md)	 - Restore from backup
* [stash restore-pvc](/docs/reference/operator/stash_restore-pvc.md)	 - Takes a restore of Persistent Volume Claim
* [stash restore-vs](/docs/reference/operator/stash_restore-vs.md)	 - Restore PVC from VolumeSnapshot
* [stash run](/docs/reference/operator/stash_run.md)	 - Launch Stash Controller
* [stash run-backup](/docs/reference/operator/stash_run-backup.md)	 - Take backup of workload paths
* [stash run-hook](/docs/reference/operator/stash_run-hook.md)	 - Execute Backup or Restore Hooks
* [stash snapshots](/docs/reference/operator/stash_snapshots.md)	 - Get snapshots of restic repo
* [stash update-status](/docs/reference/operator/stash_update-status.md)	 - Update status of Repository, Backup/Restore Session
* [stash version](/docs/reference/operator/stash_version.md)	 - Prints binary version number.

