---
title: Stash
menu:
  product_stash_0.1.0:
    identifier: stash
    name: Stash
    parent: stash-cli
    weight: 0

product_name: stash
section_menu_id: reference
menu_name: product_stash_0.1.0
url: /products/stash/0.1.0/reference/cli/
aliases:
  - /products/stash/0.1.0/reference/cli/cli/

---
## stash

kubectl plugin for Stash by AppsCode

### Synopsis

kubectl plugin for Stash by AppsCode. For more information, visit here: https://appscode.com/products/stash

### Options

```
      --alsologtostderr                  log to standard error as well as files
      --enable-analytics                 Send analytical events to Google Analytics (default true)
      --enable-status-subresource        If true, uses sub resource for crds.
  -h, --help                             help for stash
      --log-flush-frequency duration     Maximum number of seconds between log flushes (default 5s)
      --log_backtrace_at traceLocation   when logging hits line file:N, emit a stack trace (default :0)
      --log_dir string                   If non-empty, write log files in this directory
      --logtostderr                      log to standard error instead of files
      --service-name string              Stash service name. (default "stash-operator")
      --stderrthreshold severity         logs at or above this threshold go to stderr
      --use-kubeapiserver-fqdn-for-aks   if true, uses kube-apiserver FQDN for AKS cluster to workaround https://github.com/Azure/AKS/issues/522 (default true)
  -v, --v Level                          log level for V logs
      --vmodule moduleSpec               comma-separated list of pattern=N settings for file-filtered logging
```

### SEE ALSO

* [stash backup-pv](/docs/reference/stash/stash_backup-pv.md)	 - Backup persistent volume
* [stash copy-repository](/docs/reference/stash/stash_copy-repository.md)	 - Copy Repository and Secret
* [stash delete-snapshot](/docs/reference/stash/stash_delete-snapshot.md)	 - Delete a snapshot from repository backend
* [stash delete-snapshot](/docs/reference/stash/stash_delete-snapshot.md)	 - Delete a snapshot from repository backend
* [stash download](/docs/reference/stash/stash_download.md)	 - Download snapshots
* [stash download-snapshots](/docs/reference/stash/stash_download-snapshots.md)	 - Download snapshots
* [stash trigger-backup](/docs/reference/stash/stash_trigger-backup.md)	 - Trigger a backup
* [stash unlock-local-repository](/docs/reference/stash/stash_unlock-local-repository.md)	 - Unlock Restic Repository with Local Backend
* [stash unlock-repository](/docs/reference/stash/stash_unlock-repository.md)	 - Unlock Restic Repository
* [stash unlock-repository](/docs/reference/stash/stash_unlock-repository.md)	 - Unlock Restic Repository
* [stash version](/docs/reference/stash/stash_version.md)	 - Prints binary version number.

