---
title: Run-Backup
menu:
  docs_{{ .version }}:
    identifier: stash-run-backup
    name: Run-Backup
    parent: operator
product_name: stash
section_menu_id: reference
menu_name: docs_{{ .version }}
---
## stash run-backup

Take backup of workload directories

### Synopsis

Take backup of workload directories

```
stash run-backup [flags]
```

### Options

```
      --backup-configuration string   Set BackupConfiguration Name
      --enable-cache                  Specify whether to enable caching for restic (default true)
  -h, --help                          help for run-backup
      --kubeconfig string             Path to kubeconfig file with authorization information (the master location is set by the master flag).
      --master string                 The address of the Kubernetes API server (overrides any value in kubeconfig)
      --max-connections int           Specify maximum concurrent connections for GCS, Azure and B2 backend
      --metrics-enabled               Specify whether to export Prometheus metrics
      --pushgateway-url string        URL of Prometheus pushgateway used to cache backup metrics
      --secret-dir string             Directory where storage secret has been mounted
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

