---
title: Stash
menu:
  docs_{{ .version }}:
    identifier: stash
    name: Stash
    parent: operator
    weight: 0

product_name: stash
section_menu_id: reference
menu_name: docs_{{ .version }}
url: /docs/{{ .version }}/reference/operator/
aliases:
  - /docs/{{ .version }}/reference/operator/operator/

---
## stash

Stash by AppsCode - Backup your Kubernetes Volumes

### Synopsis

Stash is a Kubernetes operator for restic. For more information, visit here: https://appscode.com/products/stash

### Options

```
      --alsologtostderr                  log to standard error as well as files
      --bypass-validating-webhook-xray   if true, bypasses validating webhook xray checks
      --enable-analytics                 Send analytical events to Google Analytics (default true)
  -h, --help                             help for stash
      --log-flush-frequency duration     Maximum number of seconds between log flushes (default 5s)
      --log_backtrace_at traceLocation   when logging hits line file:N, emit a stack trace (default :0)
      --log_dir string                   If non-empty, write log files in this directory
      --logtostderr                      log to standard error instead of files (default true)
      --service-name string              Stash service name. (default "stash-operator")
      --stderrthreshold severity         logs at or above this threshold go to stderr
      --use-kubeapiserver-fqdn-for-aks   if true, uses kube-apiserver FQDN for AKS cluster to workaround https://github.com/Azure/AKS/issues/522 (default true)
  -v, --v Level                          log level for V logs
      --vmodule moduleSpec               comma-separated list of pattern=N settings for file-filtered logging
```

### SEE ALSO

* [stash backup](/docs/reference/operator/stash_backup.md)	 - Run Stash Backup
* [stash backup-pvc](/docs/reference/operator/stash_backup-pvc.md)	 - Takes a backup of Persistent Volume Claim
* [stash check](/docs/reference/operator/stash_check.md)	 - Check restic backup
* [stash create-backupsession](/docs/reference/operator/stash_create-backupsession.md)	 - create a BackupSession
* [stash create-vs](/docs/reference/operator/stash_create-vs.md)	 - Take snapshot of PersistentVolumeClaims
* [stash forget](/docs/reference/operator/stash_forget.md)	 - Delete snapshots from a restic repository
* [stash recover](/docs/reference/operator/stash_recover.md)	 - Recover restic backup
* [stash restore](/docs/reference/operator/stash_restore.md)	 - Restore from backup
* [stash restore-pvc](/docs/reference/operator/stash_restore-pvc.md)	 - Takes a restore of Persistent Volume Claim
* [stash restore-vs](/docs/reference/operator/stash_restore-vs.md)	 - Restore PVC from VolumeSnapshot
* [stash run](/docs/reference/operator/stash_run.md)	 - Launch Stash Controller
* [stash run-backup](/docs/reference/operator/stash_run-backup.md)	 - Take backup of workload paths
* [stash run-hook](/docs/reference/operator/stash_run-hook.md)	 - Execute Backup or Restore Hooks
* [stash scaledown](/docs/reference/operator/stash_scaledown.md)	 - Scale down workload
* [stash snapshots](/docs/reference/operator/stash_snapshots.md)	 - Get snapshots of restic repo
* [stash update-status](/docs/reference/operator/stash_update-status.md)	 - Update status of Repository, Backup/Restore Session
* [stash version](/docs/reference/operator/stash_version.md)	 - Prints binary version number.

