---
title: Restore-Vs
menu:
  docs_{{ .version }}:
    identifier: stash-restore-vs
    name: Restore-Vs
    parent: reference-operator
menu_name: docs_{{ .version }}
section_menu_id: reference
---
## stash restore-vs

Restore PVC from VolumeSnapshot

```
stash restore-vs [flags]
```

### Options

```
  -h, --help                      help for restore-vs
      --invoker-kind string       Kind of the restore invoker
      --invoker-name string       Name of the respective restore invoker
      --kubeconfig string         Path to kubeconfig file with authorization information (the master location is set by the master flag).
      --master string             The address of the Kubernetes API server (overrides any value in kubeconfig)
      --metrics-enabled           Specify whether to export Prometheus metrics (default true)
      --pushgateway-url string    Pushgateway URL where the metrics will be pushed
      --target-kind string        Kind of the Target
      --target-name string        Name of the Target
      --target-namespace string   Namespace of the Target
```

### Options inherited from parent commands

```
      --bypass-validating-webhook-xray   if true, bypasses validating webhook xray checks
      --use-kubeapiserver-fqdn-for-aks   if true, uses kube-apiserver FQDN for AKS cluster to workaround https://github.com/Azure/AKS/issues/522 (default true)
```

### SEE ALSO

* [stash](/docs/reference/operator/stash.md)	 - Stash by AppsCode - Backup your Kubernetes Volumes

