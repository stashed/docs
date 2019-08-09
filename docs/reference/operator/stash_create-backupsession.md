---
title: Create-Backupsession
menu:
  product_stash_v0.9.0-rc.0:
    identifier: stash-create-backupsession
    name: Create-Backupsession
    parent: operator
product_name: stash
section_menu_id: reference
menu_name: product_stash_v0.9.0-rc.0
---
## stash create-backupsession

create a BackupSession

### Synopsis

create a BackupSession

```
stash create-backupsession [flags]
```

### Options

```
      --backupsession.name string        Set BackupSession Name
      --backupsession.namespace string   Set BackupSession Namespace (default "default")
  -h, --help                             help for create-backupsession
      --kubeconfig string                Path to kubeconfig file with authorization information (the master location is set by the master flag).
      --master string                    The address of the Kubernetes API server (overrides any value in kubeconfig)
```

### Options inherited from parent commands

```
      --alsologtostderr                  log to standard error as well as files
      --bypass-validating-webhook-xray   if true, bypasses validating webhook xray checks
      --enable-analytics                 Send analytical events to Google Analytics (default true)
      --enable-status-subresource        If true, uses sub resource for crds.
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

* [stash](/docs/reference/operator/stash.md)	 - Stash by AppsCode - Backup your Kubernetes Volumes

