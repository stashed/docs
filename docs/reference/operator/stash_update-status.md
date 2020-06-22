---
title: Update-Status
menu:
  docs_{{ .version }}:
    identifier: stash-update-status
    name: Update-Status
    parent: reference-operator
menu_name: docs_{{ .version }}
section_menu_id: reference
---
## stash update-status

Update status of Repository, Backup/Restore Session

### Synopsis

Update status of Repository, Backup/Restore Session

```
stash update-status [flags]
```

### Options

```
      --backupsession string             Name of the Backup Session
  -h, --help                             help for update-status
      --kubeconfig string                Path to kubeconfig file with authorization information (the master location is set by the master flag).
      --master string                    The address of the Kubernetes API server (overrides any value in kubeconfig)
      --metrics-enabled                  Specify whether to export Prometheus metrics
      --metrics-labels strings           Labels to apply in exported metrics
      --metrics-pushgateway-url string   Pushgateway URL where the metrics will be pushed
      --namespace string                 Namespace of Backup/Restore Session (default "default")
      --output-dir string                Directory where output.json file will be written (keep empty if you don't need to write output in file)
      --prom-job-name string             Metrics job name (default "stash-prom-metrics")
      --repository string                Name of the Repository
      --restoresession string            Name of the Restore Session
      --target-kind string               Kind of the target
      --target-name string               Name of the target
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

