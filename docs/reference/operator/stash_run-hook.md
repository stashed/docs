---
title: Run-Hook
menu:
  docs_{{ .version }}:
    identifier: stash-run-hook
    name: Run-Hook
    parent: reference-operator
menu_name: docs_{{ .version }}
section_menu_id: reference
---
## stash run-hook

Execute Backup or Restore Hooks

```
stash run-hook [flags]
```

### Options

```
      --backupsession string             Name of the respective BackupSession object
  -h, --help                             help for run-hook
      --hook-type string                 Type of hook to execute
      --hostname string                  Name of the host that is being backed up or restored (default "host-0")
      --invoker-kind string              Type of the backup invoker
      --invoker-name string              Name of the respective backup invoker
      --kubeconfig string                Path to kubeconfig file with authorization information (the master location is set by the master flag).
      --master string                    The address of the Kubernetes API server (overrides any value in kubeconfig)
      --metrics-enabled                  Specify whether to export Prometheus metrics
      --metrics-labels strings           Labels to apply in exported metrics
      --metrics-pushgateway-url string   Pushgateway URL where the metrics will be pushed
      --output-dir string                Directory where output.json file will be written (keep empty if you don't need to write output in file)
      --target-kind string               Kind of the Target
      --target-name string               Name of the Target
```

### Options inherited from parent commands

```
      --bypass-validating-webhook-xray   if true, bypasses validating webhook xray checks
      --use-kubeapiserver-fqdn-for-aks   if true, uses kube-apiserver FQDN for AKS cluster to workaround https://github.com/Azure/AKS/issues/522 (default true)
```

### SEE ALSO

* [stash](/docs/reference/operator/stash.md)	 - Stash by AppsCode - Backup your Kubernetes Volumes

