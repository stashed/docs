---
title: Scaledown
menu:
  docs_{{ .version }}:
    identifier: stash-scaledown
    name: Scaledown
    parent: reference-operator
menu_name: docs_{{ .version }}
section_menu_id: reference
---
## stash scaledown

Scale down workload

```
stash scaledown [flags]
```

### Options

```
  -h, --help                help for scaledown
      --kubeconfig string   Path to kubeconfig file with authorization information (the master location is set by the master flag).
      --master string       The address of the Kubernetes API server (overrides any value in kubeconfig)
      --selector string     Label used to select Restic's workload
```

### Options inherited from parent commands

```
      --bypass-validating-webhook-xray   if true, bypasses validating webhook xray checks
      --service-name string              Stash service name. (default "stash-operator")
      --use-kubeapiserver-fqdn-for-aks   if true, uses kube-apiserver FQDN for AKS cluster to workaround https://github.com/Azure/AKS/issues/522 (default true)
```

### SEE ALSO

* [stash](/docs/reference/operator/stash.md)	 - Stash by AppsCode - Backup your Kubernetes Volumes

