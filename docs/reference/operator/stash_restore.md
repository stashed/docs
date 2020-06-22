---
title: Restore
menu:
  docs_{{ .version }}:
    identifier: stash-restore
    name: Restore
    parent: reference-operator
menu_name: docs_{{ .version }}
section_menu_id: reference
---
## stash restore

Restore from backup

### Synopsis

Restore from backup

```
stash restore [flags]
```

### Options

```
      --backoff-max-wait duration   Maximum wait for initial response from kube apiserver; 0 disables the timeout
      --enable-cache                Specify whether to enable caching for restic (default true)
  -h, --help                        help for restore
      --kubeconfig string           Path to kubeconfig file with authorization information (the master location is set by the master flag).
      --master string               The address of the Kubernetes API server (overrides any value in kubeconfig)
      --max-connections int         Specify maximum concurrent connections for GCS, Azure and B2 backend
      --metrics-enabled             Specify whether to export Prometheus metrics
      --pushgateway-url string      Pushgateway URL where the metrics will be pushed
      --restore-model string        Specify whether using job or init-container to restore (default init-container) (default "init-container")
      --restoresession string       Name of the respective RestoreSession object.
      --secret-dir string           Directory where storage secret has been mounted
```

### Options inherited from parent commands

```
      --alsologtostderr                  log to standard error as well as files
      --bypass-validating-webhook-xray   if true, bypasses validating webhook xray checks
      --enable-analytics                 Send analytical events to Google Analytics (default true)
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

