---
title: Run-Backup
menu:
  docs_{{ .version }}:
    identifier: stash-run-backup
    name: Run-Backup
    parent: reference-operator
menu_name: docs_{{ .version }}
section_menu_id: reference
---
## stash run-backup

Take backup of workload paths

```
stash run-backup [flags]
```

### Options

```
      --enable-cache             Specify whether to enable caching for restic (default true)
  -h, --help                     help for run-backup
      --host string              Name of the host that will be backed up
      --invoker-kind string      Kind of the backup invoker
      --invoker-name string      Name of the respective backup invoker
      --kubeconfig string        Path to kubeconfig file with authorization information (the master location is set by the master flag).
      --master string            The address of the Kubernetes API server (overrides any value in kubeconfig)
      --max-connections int      Specify maximum concurrent connections for GCS, Azure and B2 backend
      --metrics-enabled          Specify whether to export Prometheus metrics
      --pushgateway-url string   URL of Prometheus pushgateway used to cache backup metrics
      --target-kind string       Kind of the Target
      --target-name string       Name of the Target
```

### Options inherited from parent commands

```
      --bypass-validating-webhook-xray   if true, bypasses validating webhook xray checks
      --service-name string              Stash service name. (default "stash-operator")
      --use-kubeapiserver-fqdn-for-aks   if true, uses kube-apiserver FQDN for AKS cluster to workaround https://github.com/Azure/AKS/issues/522 (default true)
```

### SEE ALSO

* [stash](/docs/reference/operator/stash.md)	 - Stash by AppsCode - Backup your Kubernetes Volumes

