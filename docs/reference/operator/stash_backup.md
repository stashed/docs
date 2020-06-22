---
title: Backup
menu:
  docs_{{ .version }}:
    identifier: stash-backup
    name: Backup
    parent: reference-operator
menu_name: docs_{{ .version }}
section_menu_id: reference
---
## stash backup

Run Stash Backup

### Synopsis

Run Stash Backup

```
stash backup [flags]
```

### Options

```
      --burst int                The maximum burst for throttle (default 100)
      --docker-registry string   Check job image registry. (default "appscode")
  -h, --help                     help for backup
      --image-tag string         Check job image tag.
      --kubeconfig string        Path to kubeconfig file with authorization information (the master location is set by the master flag).
      --master string            The address of the Kubernetes API server (overrides any value in kubeconfig)
      --pushgateway-url string   URL of Prometheus pushgateway used to cache backup metrics
      --qps float                The maximum QPS to the master from this client (default 100)
      --restic-name string       Name of the Restic used as configuration.
      --resync-period duration   If non-zero, will re-list this often. Otherwise, re-list will be delayed aslong as possible (until the upstream source closes the watch or times out. (default 5m0s)
      --run-via-cron             Run backup periodically via cron.
      --scratch-dir emptyDir     Directory used to store temporary files. Use an emptyDir in Kubernetes. (default "/tmp")
      --workload-kind string     Kind of workload where sidecar pod is added.
      --workload-name string     Name of workload where sidecar pod is added.
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

